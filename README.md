## 心血来潮，突然想在git上记录一些东西，写一些教程，希望能帮助到有需要的人，下面记录一次Js逆向拼多多旗下社群团购sign：签名 分析加密过程
#### 前段时间接到一个客户的需求，他希望全自动把商品发布到某社群团购平台，商品是全品类，意思就是不管吃的、喝的、穿的、只要他有的全部往上发布....
#### 好了，废话不多说直接上图
- 先传个商品，看看往服务器提交了什么参数![先传个商品，看看往服务器提交了什么参数](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/1.png)  
- 找到提交参数的请求![找到提交参数的请求](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/3.png)  
- 看一下请求由哪些参数组成![看一下请求由哪些参数组成](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/4.png)  
- 直接用python模拟提交，很显然服务器验证失败了，接下来要检查一下哪些字段是每次请求都会变动的![直接用python模拟提交，很显然服务器验证失败了，接下来要检查一下哪些字段是每次请求都会变动的](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/5.png)  
- 发现一共有4个参数提交商品的时候每次都不一样，这次主要逆向这个sign参数![发现一共有4个参数提交商品的时候每次都不一样，这次主要逆向这个sign参数](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/6.png)  
- 找到该请求的调用的js文件，直接搜索参数名称![找到该请求的调用的js文件，直接搜索参数名称](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/8.png)  
- 很容易看出来sign由函数K生成出来的![很容易看出来sign由函数K生成出来的](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/9.png)  
- 找到函数K![找到函数K](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/10.png)  
- 发现函数K里面又调用了其他函数q,V,L![发现函数K里面又调用了其他函数q,V,L](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/11.png)  
- 接着往下找，先找函数q，函数q只做了简单的字符串运算改变，先不管它，接着往下找![接着往下找，先找函数q，函数q只做了简单的字符串运算改变，先不管它，接着往下找](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/12.png)  
- 找到函数V，先记住位置![找到函数V，先记住位置](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/13.png)  
- 找到函数L  ![找到函数L](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/14.png)  
- 我们再回到最初的地方

- 函数K里面的参数由变量f，c，p 三个不固定参数组成![函数K里面的参数由变量f，c，p 三个不固定参数组成](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/16.png)
- f变量取的用户id倒数4位![f变量取的用户id倒数4位](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/15.png)
- c变量取的13位时间戳![c变量取的13位时间戳](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/19.png)
- 而P变量经过反复对比，取的是本次所请求的地址后缀![而P变量经过反复对比，取的是本次所请求的地址后缀](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/18.png)

- 那么现在已经理清楚sign生成规则了，直接扣js代码

- python写个函数来调用下面的js，到这里sign参数生成出来了，经过与目标网站比对，同样时间戳生成出来的签名完全一致![图片](http://yungengxin.oss-cn-beijing.aliyuncs.com/pdd/23.png)<br>

```javascript
function L(e, t) {
        var n = e[0]
          , r = e[1]
          , o = e[2]
          , i = e[3];
        n = F(n, r, o, i, t[0], 7, -680876936),
        i = F(i, n, r, o, t[1], 12, -389564586),
        o = F(o, i, n, r, t[2], 17, 606105819),
        r = F(r, o, i, n, t[3], 22, -1044525330),
        n = F(n, r, o, i, t[4], 7, -176418897),
        i = F(i, n, r, o, t[5], 12, 1200080426),
        o = F(o, i, n, r, t[6], 17, -1473231341),
        r = F(r, o, i, n, t[7], 22, -45705983),
        n = F(n, r, o, i, t[8], 7, 1770035416),
        i = F(i, n, r, o, t[9], 12, -1958414417),
        o = F(o, i, n, r, t[10], 17, -42063),
        r = F(r, o, i, n, t[11], 22, -1990404162),
        n = F(n, r, o, i, t[12], 7, 1804603682),
        i = F(i, n, r, o, t[13], 12, -40341101),
        o = F(o, i, n, r, t[14], 17, -1502002290),
        n = U(n, r = F(r, o, i, n, t[15], 22, 1236535329), o, i, t[1], 5, -165796510),
        i = U(i, n, r, o, t[6], 9, -1069501632),
        o = U(o, i, n, r, t[11], 14, 643717713),
        r = U(r, o, i, n, t[0], 20, -373897302),
        n = U(n, r, o, i, t[5], 5, -701558691),
        i = U(i, n, r, o, t[10], 9, 38016083),
        o = U(o, i, n, r, t[15], 14, -660478335),
        r = U(r, o, i, n, t[4], 20, -405537848),
        n = U(n, r, o, i, t[9], 5, 568446438),
        i = U(i, n, r, o, t[14], 9, -1019803690),
        o = U(o, i, n, r, t[3], 14, -187363961),
        r = U(r, o, i, n, t[8], 20, 1163531501),
        n = U(n, r, o, i, t[13], 5, -1444681467),
        i = U(i, n, r, o, t[2], 9, -51403784),
        o = U(o, i, n, r, t[7], 14, 1735328473),
        n = H(n, r = U(r, o, i, n, t[12], 20, -1926607734), o, i, t[5], 4, -378558),
        i = H(i, n, r, o, t[8], 11, -2022574463),
        o = H(o, i, n, r, t[11], 16, 1839030562),
        r = H(r, o, i, n, t[14], 23, -35309556),
        n = H(n, r, o, i, t[1], 4, -1530992060),
        i = H(i, n, r, o, t[4], 11, 1272893353),
        o = H(o, i, n, r, t[7], 16, -155497632),
        r = H(r, o, i, n, t[10], 23, -1094730640),
        n = H(n, r, o, i, t[13], 4, 681279174),
        i = H(i, n, r, o, t[0], 11, -358537222),
        o = H(o, i, n, r, t[3], 16, -722521979),
        r = H(r, o, i, n, t[6], 23, 76029189),
        n = H(n, r, o, i, t[9], 4, -640364487),
        i = H(i, n, r, o, t[12], 11, -421815835),
        o = H(o, i, n, r, t[15], 16, 530742520),
        n = $(n, r = H(r, o, i, n, t[2], 23, -995338651), o, i, t[0], 6, -198630844),
        i = $(i, n, r, o, t[7], 10, 1126891415),
        o = $(o, i, n, r, t[14], 15, -1416354905),
        r = $(r, o, i, n, t[5], 21, -57434055),
        n = $(n, r, o, i, t[12], 6, 1700485571),
        i = $(i, n, r, o, t[3], 10, -1894986606),
        o = $(o, i, n, r, t[10], 15, -1051523),
        r = $(r, o, i, n, t[1], 21, -2054922799),
        n = $(n, r, o, i, t[8], 6, 1873313359),
        i = $(i, n, r, o, t[15], 10, -30611744),
        o = $(o, i, n, r, t[6], 15, -1560198380),
        r = $(r, o, i, n, t[13], 21, 1309151649),
        n = $(n, r, o, i, t[4], 6, -145523070),
        i = $(i, n, r, o, t[11], 10, -1120210379),
        o = $(o, i, n, r, t[2], 15, 718787259),
        r = $(r, o, i, n, t[9], 21, -343485551),
        e[0] = J(n, e[0]),
        e[1] = J(r, e[1]),
        e[2] = J(o, e[2]),
        e[3] = J(i, e[3])
    } 

 function z(e, t, n, r, o, i) {
        return t = J(J(t, e), J(r, i)),
        J(t << o | t >>> 32 - o, n)
    }
    function F(e, t, n, r, o, i, a) {
        return z(t & n | ~t & r, e, t, o, i, a)
    }
    function U(e, t, n, r, o, i, a) {
        return z(t & r | n & ~r, e, t, o, i, a)
    }
    function H(e, t, n, r, o, i, a) {
        return z(t ^ n ^ r, e, t, o, i, a)
    }
    function $(e, t, n, r, o, i, a) {
        return z(n ^ (t | ~r), e, t, o, i, a)
    }
    function V(e) {
        var t, n = [];
        for (t = 0; t < 64; t += 4)
            n[t >> 2] = e.charCodeAt(t) + (e.charCodeAt(t + 1) << 8) + (e.charCodeAt(t + 2) << 16) + (e.charCodeAt(t + 3) << 24);
        return n
    }
      var G = "0123456789abcdef".split("");
    function q(e) {
        for (var t = "", n = 0; n < 4; n++)
            t += G[e >> 8 * n + 4 & 15] + G[e >> 8 * n & 15];
        return t
    }

    function K(e) {
        return function(e) {
            for (var t = 0; t < e.length; t++)
                e[t] = q(e[t]);
            return e.join("")
        }(function(e) {
            var t, n = e.length, r = [1732584193, -271733879, -1732584194, 271733878];
            for (t = 64; t <= e.length; t += 64)
                L(r, V(e.substring(t - 64, t)));
            e = e.substring(t - 64);
            var o = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            for (t = 0; t < e.length; t++)
                o[t >> 2] |= e.charCodeAt(t) << (t % 4 << 3);
            if (o[t >> 2] |= 128 << (t % 4 << 3),
            t > 55)
                for (L(r, o),
                t = 0; t < 16; t++)
                    o[t] = 0;
            return o[14] = 8 * n,
            L(r, o),
            r
        }(e))
    }

      function J(e, t) {
        return e + t & 4294967295
    } 

