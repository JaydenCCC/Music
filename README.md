# Music

### 一、播放器基本功能
- [x] 歌曲播放、切歌、进度控制
- [x] 三种播放模式
- [x] 搜索歌手、专辑、歌曲
- [x] 上拉加载功能、延迟加载
- [x] Mini播放器控制
- [x] 收藏歌曲

### 二、项目结构

```text
│  App.vue                  // 组件入口
│  main.js                  // js入口
│  
├─api                       // 获取接口数据的文件夹
│      config.js                // 公共配置
│      lyric.js                 // 歌词数据
│      rank.js                  // 排行榜数据
│      rankDetail.js            // 榜单详情数据
│      recommend.js             // 推荐页面相关数据
│      search.js                // 热门搜索
│      singer.js                // 歌手数据
│      
├─base                      // 公用基础组件文件夹
│          
├─common                    // 样式、字体、js工具库文件夹
│          
├─components                // 业务组件文件夹
│          
├─router                    // 路由配置文件夹
│      index.js
│      
└─store                     // vuex配置文件夹
        actions.js              // dispatch
        getters.js              // 计算state数据
        index.js                // vuex入口
        mutations-types.js      // mutations常量
        mutations.js            // commit
        state.js                // 基础数据
```

### 三、个人笔记
#### 后端接口代理

```
以前在 dev.server.js 里用 express 框架搭了一个服务器，设置一个接口，然后让前端去请求这个接口，
在这个接口里再去请求QQ音乐的歌单接口并且设置相应的头文件信息如referer,host，这样就达到了目的。

由于现在vue-cli搭出来的项目结构不一样，没有dev.server.js文件，所以这里采取了两种可行的方式。
在 config 文件夹下的 index.js 里提供了一个 proxyTable 属性，在这里设置代理是最简单的方法。
```

- ##  两种方式
> 1、build 文件夹下的 webpack.dev.conf.js

```
// 获取推荐歌单 前端代码
export function getDiscList() {
    // http://localhost:8080/api/getDiscList 触发后端接口代理, 再访问真正的目标 url: https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg
    const url = '/api/getDiscList'

    const data = Object.assign({}, commonParams, {
        platform: 'yqq',
        hostUin: 0,
        sin: 0,
        ein: 29,
        sortId: 5,
        needNewCode: 0,
        categoryId: 10000000,
        rnd: Math.random(),
        // 这次的请求是 ajax 请求, 需要 respnse 是json格式
        format: 'json'
    })

    return axios.get(url, {
        // 后端接口代理中 express 可以通过 request.query 拿到 data
        params: data
    }).then(res => {
        // 返回一个Promise
        return Promise.resolve(res.data)
    })
}


// 在 webpack.dev.conf.js 中实现后端接口代理
// 第一步
var express = require('express')
var axios = require('axios')
//app 是 express 的实例
var app = express()
var apiRoutes = express.Router()
app.use('/api', apiRoutes)

// 第二步 在 devServer 里面添加 before 方法, 逗号别漏了
    before(app) {
    
      // app.get 的第二个参数是函数, 通过该函数的第一个参数 request.query 来得到请求时提交的数据 data
      app.get('/api/getDiscList', function (req, res) {
        const url = 'https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg'
        // 使用 axios 访问真正的目标 url
        axios.get(url, {
          // 就是为了在此改变 headers 才需要的代理
          headers: {
            referer: 'https://c.y.qq.com/',
            host: 'c.y.qq.com'
          },
          params: req.query
        }).then(response => {
          // 这里的 response 是请求成功时qq音乐返回的数据, 需要用 res 向前端传递, 
          // 且需要传递的是 response.data, 如果传递 response 会报错
          res.json(response.data)
        }).catch(err => {
          console.log(err)
        })
      })
    }
```

---

> 2、config 文件夹下的 index.js

```
// 获取推荐歌单 前端代码不变
export function getDiscList() {
    const url = '/api/getDiscList'

    const data = Object.assign({}, commonParams, {
        platform: 'yqq',
        hostUin: 0,
        sin: 0,
        ein: 29,
        sortId: 5,
        needNewCode: 0,
        categoryId: 10000000,
        rnd: Math.random(),
        format: 'json'
    })

    return axios.get(url, {
        params: data
    }).then(res => {
        return Promise.resolve(res.data)
    })
}


// 在 config 文件夹下的 index.js 找到 proxyTable: {} 填写代理配置
proxyTable: {
  '/api/getDiscList': {
    // target属性是真实请求的目标 url
    target: 'https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg',
    // bypass 对应的函数是请求之前可以进行的操作, req 这个参数就是请求的信息，可以设置请求头信息, 这里是修改 referer 和 host
    bypass: function (req, res, proxyOptions) {
      req.headers.referer = 'https://c.y.qq.com/';
      req.headers.host = 'c.y.qq.com';
    },
    // 通过 pathRewrite 设置一下前缀，最后在前端请求的接口写成设置的接口名('/api/getDiscList')就能进行请求了
    pathRewrite: {
      '^/api/getDiscList': ''
    }
  }
},

```
