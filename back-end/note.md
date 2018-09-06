#图片抓取示例，来自炮总
```js
const express = require('express')
const mkdirp = require('mkdirp')
const charset = require('superagent-charset')
const superagent = charset(require('superagent'))
const cheerio = require('cheerio')
const request = require('request')
const async = require('async')
const fs = require('fs')
const _ = require('lodash')
const app = express()
let options = {
    url: 'http://www.notmaya.com/viewthread.php?tid=2151933&extra=page%3D1',
    dir: './image/', // 保存目录
    downLimit: 2    // 图片并行下载上限
}
const getCookie = function () { // 获取maya的cookie
    return new Promise(function (resolve) {
        superagent.post('http://www.notmaya.com/logging.php?action=login')
            .type('form')
            .send({
                formhash: '42c3130e',
                referer: 'index.php',
                cookietime: '2592000',
                loginfield: 'username',
                username: '',
                password: '',
                questionid: '0',
                answer: '',
                loginmode: '',
                styleid: '',
                loginsubmit: '(unable to decode value)'
            })
            .end(function (err, res) {
                if (err) {
                    handleErr(err.message);
                    return;
                }
                resolve(res.header['set-cookie']) // 从response中得到cookie
            })
    })
}
// 获取到页面的charset并塞入cookie,获取到跳转后的页面的html
const parseUrl = function (url, cookie, callback) {
    async.waterfall([
        function (callback) {   // 动态获取网站编码
            superagent.get(url).end(function (err, res) {
                var charset = "utf-8";
                var arr = res.text.match(/<meta([^>]*?)>/g);
                if (arr) {
                    arr.forEach(function (val) {
                        var match = val.match(/charset\s*=\s*(.+)\"/);
                        if (match && match[1]) {
                            if (match[1].substr(0, 1) == '"') match[1] = match[1].substr(1);
                            charset = match[1].trim();
                        }
                    })
                }
                callback(err, charset)
            })
        }, function (charset, callback) {   // 内容爬取
            superagent
                .get(url)
                .charset(charset)
                .set('Cookie', cookie)
                .end(function (err, res) {
                    if (err) {
                        console.log(err);
                        callback(err);
                        return;
                    }
                    var model = {};
                    var $ = cheerio.load(res.text);
                    var title = _.trim($('title').text());
                    if (title.indexOf('-') > 0) {
                        var strs = _.split(title, '-');
                        model.title = _.trim(title.substr(0, title.lastIndexOf('-')));
                        model.source = _.trim(_.last(strs));
                    } else {
                        model.title = _.trim(title);
                    }
                    model.html = res.text
                    callback(err, model);
                })
        }
    ],
        function (err, model) {
            callback(err, model);
        });
}
// 下载图片方法
const download = function (src, dir, filename) {
    // async.eachLimit()
    request.head(src, function (err, res, body) {
        // request(url).pipe(fs.createWriteStream(dir + '/' + filename));
        request.get(src).pipe(fs.createWriteStream(dir + '/' + Date.now() + src.substr(-4, 4))).on('finish', function () {
            console.log('finish: ' + src)
        })
    });
};

app.get('/index', function (req, res) {
    // superagent
    let imageList = []
    mkdirp(options.dir, function (err) {
        if (err) {
            console.log(err)
        }
    })
    getCookie().then(function (cookie) {
        console.log(cookie)
        parseUrl(options.url, cookie, function (err, model) {
            let $ = cheerio.load(model.html)
            // console.log(err)
            if (!err) {
                // 遍历html节点中的img标签
                $('.spaceborder').find('img').each(function (index, item) {
                    // console.log($(item).attr('src'))
                    let src = $(item).attr('src')
                    if (!src.includes('gif')) { // 过滤gif图片
                        imageList.push({ src: src, alt: $(item).attr('alt') })
                        download(src, options.dir)
                    }
                })
                res.json({ code: '200', imageList: imageList })
            } else {
                res.json({ code: '500' })
            }
        })
    })
})

const server = app.listen('8001', function () {
    const host = server.address().address
    const port = server.address().port
    console.log("应用实例，访问地址为 http://%s:%s", host, port)
})


```
