+++
date = '2026-07-01T08:55:47+08:00'
draft = false
title = 'Axios配置'
+++

``` javascript
import axios from 'axios'
import qs from 'qs'

switch (process.env.NODE_ENV){
  case 'production':
    axios.defaults.baseURL = 'https://api.zhufengpeixun.cn'
    break;
  case 'test':
    axios.defaults.baseURL = 'http://192.168.20.12:8080'
    break;
  default:
    axios.defaults.baseURL = 'http://127.0.0.1:3000'
}

// 设置超时请求时间
axios.defaults.timeout = 10000;

// 设置CORS 跨域允许携带资源凭证
axios.defaults.withCredentials = true

// 设置POST请求头：告知服务器请求主体的数据格式
axios.defaults.headers['Content-Type'] = 'application/x-www-form-url'

axios.defaults.transformRequest = data => qs.stringify(data)

// 设置请求拦截器
axios.interceptors.request.use(
  config => {
    // 请求开始前，取消掉当前正在进行的相同请求
    removePending(config)

    addPending(config)
    const token = localstorage.getItem('token')
    token && (config.headers.Authorization = token)

    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// 设置响应拦截器
axios.defaults.validateStatus = status => {
  return /^(2|3)\d{2}$/.test(status)
}

axios.interceptors.response.use(
  response => {
    removingPending(response.config);
    return response.data
  },
  error => {
    if(error.response){
      switch(error.response.status){
        case 400:
          error.message = '请求错误(400)';
          break;
        case 401:
          error.message = '未授权，请登录(401)';
          break;
        case 403:
          error.message = '拒绝访问(403)';
          localstorage.removeItem('token');
          break;
      }
      return Promise.reject(error.message)
    }else{
      // 断网处理
      if(!window.navigator.onLine){
        // todo: 断开网络，可以让其跳转到断网页面
        return
      }
      return Promise.reject(error)
    }
  }
)

const pendingMap = new Map()

function getPendingKey(config) {
  const {url, method, params, data} = config;
  
  let {data: requestBody} = data || {}
  
  if(typeof requestBody === 'string'){
    requestBody = JSON.parse(requestBody)
  }
  return [url, method, JSON.stringify(params), JSON.stringify(requestBody)].concat('&')
}

function addPending(config){
  const pendingKey = getPendingKey(config)
  
  config.cancelToken = config.cancelToken || new axios.CancelToken((cancel) => {
    if(!pendingMap.has(pendingKey)){
      pendingMap.set(pendingKey, cancel)
    }
  })
}

function removePending(config){
  const pendingKey = getPendingKey(config);
  
  if(pendingMap.has(pendingKey)){
    const cancel = pengingMap.get(pendingKey);
    cancel(pendingKey);
    pendingMap.delete(pendingKey);
  }
}

function clearPending(){
  for(const [url, cancel] of pendingMap){
    cancel(url)
  }
  
  pendingMap.clear();
}

export default axios

```