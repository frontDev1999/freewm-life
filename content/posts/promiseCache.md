+++
date = '2026-07-01T09:04:40+08:00'
draft = false
title = 'PromiseCache'
+++

## 使用场景

> 多次触发，只执行一次

#### 多个组件请求同一个接口

``` javascript

const pending = new Map();

function dedupRequest(key, requestFn) {

    if(pending.has(key)){
        return pending.get(key)
    }

    const p = requestFn().finally(() => {
        pending.delete(key)
    })

    pending.set(key, p)

    return p
}

// example

dedupRequest('user-info', () => axios.get("/user"))
```

#### 按钮防重复提交

``` javascript

let submitPromise = null;

async function handleSubmit(data) {

    if(submitPromise) return submitPromise;

    submitPromise = axios.post("/order", data).finally(() => {
        submitPromise = null
    })
    return submitPromise
}

```

#### 多次触发，只保留最后一次

``` javascript

let currentRequestId = 0;

async function search(keyword){

    const id = ++ currentRequestId;

    const res = await axios.get('/search', {params: {q: keyword}})

    if(id !== currentRequestId) return

    setResult(res.data)
}

// 加强版

let controller = null;

async function search(keyword){

    controller?.abort();

    controller = new AbortController();

    const res = await axios.get('/search', {
        params: {q: keyword},
        signal: controller.signal
    })

    setResult(res.data)
}

```