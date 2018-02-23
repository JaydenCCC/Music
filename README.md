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

- #####  两种方式
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

#### 生命周期
```
   //组件实例创建前触发
    beforeCreate: function() {
        console.log(this.$data);
        console.log(this.$el);
    }, 
    //组件实例创建后触发,属性已绑定,但DOM还未生成, $el 属性还不存在
    created: function() {
        console.log(this.$data);
        console.log(this.$el);
    }, 
    //挂载前
    beforeMount: function() {
        console.log(this.$data);
        console.log(this.$el);
    },
    //挂载结束
    mounted: function() {
        console.log(this.$data);
        console.log(this.$el);
    },
    //数据更新前
    beforeUpdate: function() {},
    updated: function() {},
    //数据更新后
    //组件销毁前调用
    beforeDestroy: function() {},
    //组件销毁后调用
    destroyed: function() {}
    
    在 created 中放后端请求比较好,这时候DOM还没开始渲染,等把数据请求回来了一起渲染
    在 mounted 中可以是对插件的初始化,例如 better-scroll 需要对父元素和子元素的高度之差做计算来设置可以滚动的位置,
    所以实例化或者调用 refresh() 的时候一定要保证 DOM 是渲染完成的
    mounted 里设置 setTimeout 的时间为20 的原因是: 浏览器刷新一次的时间为17毫秒
```

####  封装方法
> 

- 使用: 在 mounted 钩子调用的方法中使用 addClass 给 el 添加类名
```
export function hasClass(el, className) {
  let reg = new RegExp('(^|\\s)' + className + '(\\s|$)')
  return reg.test(el.className)
}

export function addClass(el, className) {
  if (hasClass(el, className)) {
    return
  }

  // el.className 是以空格隔开各个类名
  let newClass = el.className.split(' ')
  newClass.push(className)
  el.className = newClass.join(' ')
}
```
<br />

- 在 script 里面设置dom元素的样式,我们需要兼容各种浏览器，为了不需要重复写代码，在 dom.js 里面封装了相应的方法
```
// 能力检测
let elementStyle = document.createElement('div').style

let vendor = (() => {
  let transformNames = {
    webkit: 'webkitTransform',
    Moz: 'MozTransform',
    O: 'OTransform',
    ms: 'msTransform',
    standard: 'transform'
  }

  for (let key in transformNames) {
    if (elementStyle[transformNames[key]] !== undefined) {
      return key
    }
  }

  return false
})()

export function prefixStyle(style) {
  if (vendor === false) {
    return false
  }

  if (vendor === 'standard') {
    return style
  }

  return vendor + style.charAt(0).toUpperCase() + style.substr(1)
}


// 如何使用
const transform = prefixStyle('transform')

this.$refs.layer.style[transform]=`translate3d(0,${translateY}px,0)`
```

<br />

- 数组洗牌
```
// 返回随机整数
function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1) + min)
}

// 为了不修改原数组，需要对原数组的副本进行操作
export function shuffle(arr) {
    let _arr = arr.slice()
    for (let i = 0; i < _arr.length; i++) {
        let j = getRandomInt(0, i)
        
        let temp = _arr[i]
        _arr[i] = _arr[j]
        _arr[j] = temp
    }
    return _arr
}
```
<br />

- 节流
```
// 节流
export function debounce(func, delay) {
    let timer
    return function (...args) {
        if (timer) {
            clearTimeout(timer)
        }
        timer = setTimeout(() => {
            func.apply(this, args)
        }, delay)
    }
}

// 未使用节流时
created() {
  this.$watch('query', (newQuery) => {
    this.$emit('query', newQuery)
  })
}

// 使用节流
created() {
  this.$watch('query', debounce((newQuery) => {
    this.$emit('query', newQuery)
  },200))
}
```
<br />

- localstorage插件使用, 和 findIndex 的另一种用法
```
import storage from 'good-storage'

const SEARCH_KEY = '__search__'
const SEARCH_MAX_LENGTH = 15

function insertArray(arr, val, compare, maxLen) { // 定义一个比较函数 compare
    const index = arr.findIndex(compare)
    if (index === 0) {
        return
    }
    if (index > 0) {
        arr.splice(index, 1)
    }
    arr.unshift(val)
    if (maxLen && arr.length > maxLen) {
        // 超过数组最大长度,则踢出最后一个
        arr.pop()
    }
}

// 保存搜索历史
export function saveSearch(query) {
    let searches = storage.get(SEARCH_KEY, []) // 第二个参数表示默认值
    insertArray(searches, query, (item) => {
        return item === query
    }, SEARCH_MAX_LENGTH)

    storage.set(SEARCH_KEY, searches)
    return searches
}
```
