---
layout: post
title: "elasticsearch滚动查询"
date: 2023-06-06
tags: [elasticsearch]
---
# elasticsearch滚动查询
es深翻页默认最大10000条,修改maxResult后存在性能问题,容易导致es崩溃
部分业务场景可以使用滚动查询
## 封装滚动查询工具类
```java
/**
 * @author Q1sj
 * @date 2023.3.27 13:45
 */
@Slf4j
public class EsScrollSearch<T> implements Iterable<List<T>> {
    private final ElasticsearchRestTemplate elasticsearchRestTemplate;
    private final Query query;
    private final Class<T> clazz;

    private final IndexCoordinates index;

    private final long scrollTimeInMillis;

    private final EsScrollSearchIterator iterator;
    /**
     * 上次查询结果
     */
    private SearchScrollHits<T> lastSearchScrollHits;
    private String scrollId;

    private int alreadySearchCount = 0;

    public EsScrollSearch(ElasticsearchRestTemplate elasticsearchRestTemplate, NativeSearchQuery query, Class<T> clazz, IndexCoordinates index) {
        this(elasticsearchRestTemplate, (Query) query, clazz, index);
        if (query.getMaxResults() == null) {
            query.setMaxResults(10000);
        }
    }

    public EsScrollSearch(ElasticsearchRestTemplate elasticsearchRestTemplate, CriteriaQuery query, Class<T> clazz, IndexCoordinates index) {
        this(elasticsearchRestTemplate, (Query) query, clazz, index);
        if (query.getMaxResults() == null) {
            query.setMaxResults(10000);
        }
    }

    private EsScrollSearch(ElasticsearchRestTemplate elasticsearchRestTemplate, Query query, Class<T> clazz, IndexCoordinates index) {
        this.elasticsearchRestTemplate = elasticsearchRestTemplate;
        this.query = query;
        this.clazz = clazz;
        this.index = index;
        this.iterator = new EsScrollSearchIterator();
        this.scrollTimeInMillis = 60 * 1000;
    }

    /**
     * 获取所有数据
     * 超大数据使用迭代器分批处理防止oom
     *
     * @return
     */
    public List<T> getAllData() {
        long start = System.currentTimeMillis();
        List<T> all = new ArrayList<>();
        try {
            for (List<T> list : this) {
                all.addAll(list);
            }
        } finally {
            clear();
        }
        log.debug("共计查询:{}条,耗时:{}ms", all.size(), System.currentTimeMillis() - start);
        return all;
    }

    public void clear() {
        log.debug("{} clear", index.getIndexName());
        if (scrollId == null) {
            return;
        }
        // 清除 scroll
        elasticsearchRestTemplate.searchScrollClear(Collections.singletonList(scrollId));
        alreadySearchCount = 0;
        scrollId = null;
    }

    @Override
    public Iterator<List<T>> iterator() {
        return iterator;
    }

    private class EsScrollSearchIterator implements Iterator<List<T>> {
        @Override
        public boolean hasNext() {
            return scrollId == null || lastSearchScrollHits.hasSearchHits();
        }

        @Override
        public List<T> next() {
            if (scrollId == null) {
                return firstQuery();
            }
            if (lastSearchScrollHits != null && !lastSearchScrollHits.hasSearchHits()) {
                return Collections.emptyList();
            }
            // 后续查询
            SearchScrollHits<T> searchScrollHits = elasticsearchRestTemplate.searchScrollContinue(scrollId, scrollTimeInMillis, clazz, index);
            int size = searchScrollHits.getSearchHits().size();
            alreadySearchCount += size;
            log.debug("{}共:{}条 已查询:{} 本次:{}", index.getIndexName(), searchScrollHits.getTotalHits(), alreadySearchCount, size);
            lastSearchScrollHits = searchScrollHits;
            List<T> list = new ArrayList<>();
            for (SearchHit<T> searchScrollHit : searchScrollHits) {
                list.add(searchScrollHit.getContent());
            }
            return list;
        }

        private List<T> firstQuery() {
            List<T> list = new ArrayList<>();
            // 第一次查询
            SearchScrollHits<T> searchScrollHits = elasticsearchRestTemplate.searchScrollStart(scrollTimeInMillis, query, clazz, index);
            int size = searchScrollHits.getSearchHits().size();
            log.debug("{}共:{}条 本次:{}", index.getIndexName(), searchScrollHits.getTotalHits(), size);
            alreadySearchCount = size;
            if (!searchScrollHits.hasSearchHits()) {
                clear();
            }
            lastSearchScrollHits = searchScrollHits;
            scrollId = searchScrollHits.getScrollId();
            for (SearchHit<T> searchScrollHit : searchScrollHits) {
                list.add(searchScrollHit.getContent());
            }
            return list;
        }
    }
}
```