# Infinte-Scroll-in-React


# Custom Hook for InfinteScroll
```

import React, { useState, useEffect } from "react";
import axios from "axios";

const InfiniteScroll = (query, pageNumber) => {
  const [isLoading, setIsLoading] = useState(true);
  const [isError, setIsError] = useState(false);
  const [videos, setVideos] = useState([]);
  const [hasMore, setHasMore] = useState(false);

  useEffect(() => {
    setVideos([]);
  }, [query]);

  useEffect(() => {
    console.log(videos);
  }, [videos]);
  useEffect(() => {
    setIsLoading(true);
    setIsError("");

    let cancle;

    axios({
      method: "GET",
      url: "http://openlibrary.org/search.json",
      params: { q: query, page: pageNumber },
      cancelToken: new axios.CancelToken((c) => (cancle = c)),
    })
      .then((res) => {
        setVideos((preVid) => {
          return [
            ...new Set([
              ...preVid,
              ...res.data.docs.map((item, index) => {
                return {
                  title: item.title,
                  id: index,
                };
              }),
            ]),
          ];
        });
        setHasMore(res.data.docs.length > 0);
        setIsLoading(false);
      })
      .catch((err) => {
        if (axios.isCancel()) return;
      });

    return () => cancle();
  }, [query, pageNumber]);

  return {
    isLoading,
    hasMore,
    videos,
    isError,
  };
};

export default InfiniteScroll;



```

# Implementation 

```
import React, { useState, useEffect, useRef, useCallback } from "react";

// import useInfinite from './useInfinite';
import InfiniteScroll from "./InfiniteScroll";

import "./temp.css";

export const Temp = () => {
  
  const [query, setQuery] = useState("");
  const [pageNumber, setPageNumber] = useState(1);


  const { isLoading, isError, videos, hasMore } = InfiniteScroll(
    query,
    pageNumber
  );

  const observer = useRef();

  const lastVideoRef = useCallback(
    (node) => {
      if (isLoading) return;
      if (observer.current) observer.current.disconnect();

      observer.current = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting && hasMore) {
          console.log("last node", node);
          setPageNumber((prevNum) => {
            return prevNum + 1;
          });
        }
      });

      if (node) observer.current.observe(node);
    },
    [hasMore, isLoading]
  );

  const handleQueryChange = (e) => {
    setQuery(e.target.value);
    setPageNumber(1);
  };

  return (
    <>
      <div className="container p-5">
        <div className="box">
          <h1 className="text-center text-muted">Search Api</h1>

          <br />
          <div className="form-group">
            <input
              type="text"
              className="form-control"
              onChange={handleQueryChange}
              value={query}
            />
          </div>

          <div className="box scro shadow p-5 bg-dark text-white">
            {isError && <h1>Errror ::: {isError}</h1>}

            {videos.map((vid, index) => {
              if (videos.length === index + 1) {
                return (
                  <div ref={lastVideoRef} key={vid.id}>
                    {vid.title}
                  </div>
                );
              } else {
                return <div key={vid.id}>{vid.title}</div>;
              }
            })}
          </div>
          {isLoading && <h1>Loading...</h1>}
        </div>
      </div>
    </>
  );
};

export default Temp;


```

