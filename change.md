代码修改：
1. node_modules\hexo-generator-random\random.html

```js
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
    <title>每日博客跳转中...</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.0.0/crypto-js.min.js"></script>
</head>
<body onload="dogo();">
    <script>
    function dogo(){
        var urls=[
        {% for post in posts %}
        "{{ post.permalink | uriencode }}",
        {% endfor %}
        ];

        var filteredUrls = urls.filter(function(url) {
            return url.includes('/posts/') && !url.includes('/posts/summary.html');
        });

        var currentDate = new Date();
        var currentYear = currentDate.getFullYear();
        var currentMonth = currentDate.getMonth() + 1;
        var currentDay = currentDate.getDate();
        // var seconds = currentDate.getSeconds();

        var formattedDate = `${currentYear}-${currentMonth}-${currentDay}`;

        var cacheDate = 'hexo-cache-date';
        var cacheKey = 'hexo-cache-random';
        var localCacheDate = localStorage.getItem(cacheDate);
        var randomNumber = localStorage.getItem(cacheKey);

        if ((formattedDate != localCacheDate) || (randomNumber === null)) {
            var hash = CryptoJS.SHA256(formattedDate).toString();

            // var random = parseInt(hash.slice(0, 8), 16) / 0xffffffff;
            // var randomNumber = Math.floor(random * filteredUrls.length);
            randomNumber = parseInt(hash.slice(0, 8), 16) % filteredUrls.length;

            localStorage.setItem(cacheDate, formattedDate);
            localStorage.setItem(cacheKey, randomNumber);
        }

        // console.log(randomNumber);
        location.href = filteredUrls[randomNumber % filteredUrls.length];
    }
    </script>
</body>
</html>
```

``html
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width">
    <meta name="theme-color" content="#222">
    <meta name="generator" content="Hexo 6.3.0">

    <link rel="apple-touch-icon" sizes="180x180" href="/images/apple-touch-icon-next.png">
    <link rel="icon" type="image/png" sizes="32x32" href="/images/favicon-32x32-next.png">
    <link rel="icon" type="image/png" sizes="16x16" href="/images/favicon-16x16-next.png">
    <link rel="mask-icon" href="/images/logo.svg" color="#222">

    <link rel="stylesheet" href="/css/main.css">

    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"
        integrity="sha256-HtsXJanqjKTc8vVQjO4YMhiqFoXkfBsjBWcX91T1jr8=" crossorigin="anonymous">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.1.1/animate.min.css"
        integrity="sha256-PR7ttpcvz8qrF57fur/yAx1qXMFJeJFiA6pSzWi0OIE=" crossorigin="anonymous">

    <script class="next-config" data-name="main"
        type="application/json">{"hostname":"pursue26.github.io","root":"/","images":"/images","scheme":"Pisces","darkmode":false,"version":"8.18.0","exturl":false,"sidebar":{"position":"left","display":"post","padding":18,"offset":12},"copycode":{"enable":true,"style":null},"fold":{"enable":true,"height":300},"bookmark":{"enable":false,"color":"#222","save":"auto"},"mediumzoom":false,"lazyload":false,"pangu":false,"comments":{"style":"tabs","active":null,"storage":true,"lazyload":false,"nav":null},"stickytabs":false,"motion":{"enable":true,"async":false,"transition":{"menu_item":"fadeInDown","post_block":"fadeIn","post_header":"fadeInDown","post_body":"fadeInDown","coll_header":"fadeInLeft","sidebar":"fadeInUp"}},"prism":false,"i18n":{"placeholder":"搜索...","empty":"没有找到任何搜索结果：${query}","hits_time":"找到 ${hits} 个搜索结果（用时 ${time} 毫秒）","hits":"找到 ${hits} 个搜索结果"},"path":"/search.xml","localsearch":{"enable":true,"trigger":"auto","top_n_per_article":1,"unescape":false,"preload":false}}</script>
    <script src="/js/config.js"></script>

    <meta property="og:type" content="website">
    <meta property="og:title" content="每日博客">
    <meta property="og:url" content="https://pursue26.github.io/random.html">
    <meta property="og:site_name" content="aha&#39;s blog">
    <meta property="og:locale" content="zh_CN">
    <meta property="article:author" content="aha">
    <meta name="twitter:card" content="summary">

    <link rel="canonical" href="https://pursue26.github.io/random.html">

    <script class="next-config" data-name="page"
        type="application/json">{"sidebar":"","isHome":false,"isPost":false,"lang":"zh-CN","comments":true,"permalink":"https://pursue26.github.io/random.html","path":"random.html","title":"每日博客"}</script>

    <script class="next-config" data-name="calendar" type="application/json">""</script>
    <title>每日博客 | aha's blog
    </title>

    <noscript>
        <link rel="stylesheet" href="/css/noscript.css">
    </noscript>
    <style>
        .darkmode--activated {
            --body-bg-color: #282828;
            --content-bg-color: #333;
            --card-bg-color: #555;
            --text-color: #ccc;
            --blockquote-color: #bbb;
            --link-color: #ccc;
            --link-hover-color: #eee;
            --brand-color: #ddd;
            --brand-hover-color: #ddd;
            --table-row-odd-bg-color: #282828;
            --table-row-hover-bg-color: #363636;
            --menu-item-bg-color: #555;
            --btn-default-bg: #222;
            --btn-default-color: #ccc;
            --btn-default-border-color: #555;
            --btn-default-hover-bg: #666;
            --btn-default-hover-color: #ccc;
            --btn-default-hover-border-color: #666;
            --highlight-background: #282b2e;
            --highlight-foreground: #a9b7c6;
            --highlight-gutter-background: #34393d;
            --highlight-gutter-foreground: #9ca9b6
        }

        .darkmode--activated img {
            opacity: .75
        }

        .darkmode--activated img:hover {
            opacity: .9
        }

        .darkmode--activated code {
            color: #69dbdc;
            background: 0 0
        }

        button.darkmode-toggle {
            z-index: 9999
        }

        .darkmode-ignore,
        img {
            display: flex !important
        }

        .beian img {
            display: inline-block !important
        }
    </style>
</head>

<body itemscope itemtype="http://schema.org/WebPage" class="use-motion">
    <div class="headband"></div>

    <main class="main">
        <div class="column">
            <header class="header" itemscope itemtype="http://schema.org/WPHeader">
                <div class="site-brand-container">
                    <div class="site-nav-toggle">
                        <div class="toggle" aria-label="切换导航栏" role="button">
                            <span class="toggle-line"></span>
                            <span class="toggle-line"></span>
                            <span class="toggle-line"></span>
                        </div>
                    </div>

                    <div class="site-meta">

                        <a href="/" class="brand" rel="start">
                            <i class="logo-line"></i>
                            <p class="site-title">aha's blog</p>
                            <i class="logo-line"></i>
                        </a>
                    </div>

                    <div class="site-nav-right">
                        <div class="toggle popup-trigger" aria-label="搜索" role="button">
                            <i class="fa fa-search fa-fw fa-lg"></i>
                        </div>
                    </div>
                </div>



                <nav class="site-nav">
                    <ul class="main-menu menu">
                        <li class="menu-item menu-item-home"><a href="/" rel="section"><i
                                    class="fa fa-home fa-fw"></i>首页</a></li>
                        <li class="menu-item menu-item-dailyblog"><a href="/random.html" rel="section"><i
                                    class="fa fa-file fa-fw"></i>每日博客</a></li>
                        <li class="menu-item menu-item-about"><a href="/about/" rel="section"><i
                                    class="fa fa-user fa-fw"></i>关于</a></li>
                        <li class="menu-item menu-item-tags"><a href="/tags/" rel="section"><i
                                    class="fa fa-tags fa-fw"></i>标签</a></li>
                        <li class="menu-item menu-item-categories"><a href="/categories/" rel="section"><i
                                    class="fa fa-th fa-fw"></i>分类</a></li>
                        <li class="menu-item menu-item-archives"><a href="/archives/" rel="section"><i
                                    class="fa fa-archive fa-fw"></i>归档</a></li>
                        <li class="menu-item menu-item-search">
                            <a role="button" class="popup-trigger"><i class="fa fa-search fa-fw"></i>搜索
                            </a>
                        </li>
                    </ul>
                </nav>



                <div class="search-pop-overlay">
                    <div class="popup search-popup">
                        <div class="search-header">
                            <span class="search-icon">
                                <i class="fa fa-search"></i>
                            </span>
                            <div class="search-input-container">
                                <input autocomplete="off" autocapitalize="off" maxlength="80" placeholder="搜索..."
                                    spellcheck="false" type="search" class="search-input">
                            </div>
                            <span class="popup-btn-close" role="button">
                                <i class="fa fa-times-circle"></i>
                            </span>
                        </div>
                        <div class="search-result-container no-result">
                            <div class="search-result-icon">
                                <i class="fa fa-spinner fa-pulse fa-5x"></i>
                            </div>
                        </div>

                    </div>
                </div>

            </header>
        </div>

        <div class="main-inner page posts-expand">
            <div id="blogList"></div>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.0.0/crypto-js.min.js"></script>
            <script>
                function generateBlogList() {
                    var blogData = [
                        {% for post in posts %}
                {
                    "name": "{{ post.title | striptags | escape }}",
                        "url": "{{ post.permalink | uriencode }}",
                            "date": "{{ post.date }}"
                },
                {% endfor %}
                ];

                var blogData = blogData.sort(function (a, b) {
                    var dateA = new Date(parseInt(a.date));
                    var dateB = new Date(parseInt(b.date));
                    return dateA - dateB;
                });

                function compareDates(date1, date2) {
                    const year1 = date1.getFullYear();
                    const month1 = date1.getMonth();
                    const day1 = date1.getDate();
                    const year2 = date2.getFullYear();
                    const month2 = date2.getMonth();
                    const day2 = date2.getDate();

                    if (year1 < year2) {
                        return 0;
                    } else if (year1 > year2) {
                        return 1;
                    }

                    if (month1 < month2) {
                        return 0;
                    } else if (month1 > month2) {
                        return 1;
                    }

                    if (day1 < day2) {
                        return 0;
                    } else if (day1 > day2) {
                        return 1;
                    }
                    return 0;
                }

                function filterBlogData(blogData, specifiedDate) {
                    var filteredBlogData = blogData.filter(function (blog) {
                        var blogDate = new Date(parseInt(blog.date));
                        return blog.url.includes('/posts/') && !blog.url.includes('/posts/summary.html') && compareDates(blogDate, specifiedDate) === 0;
                    });
                    return filteredBlogData;
                }

                var currentDate = new Date();
                var year = currentDate.getFullYear();
                var month = currentDate.getMonth() + 1;
                var day = currentDate.getDate();

                for (var i = 0; i <= 30; i++) {
                    var pastDate = new Date(currentDate.getTime() - i * 24 * 60 * 60 * 1000);
                    var year = pastDate.getFullYear();
                    var month = pastDate.getMonth() + 1;
                    var day = pastDate.getDate();
                    var weekday = pastDate.getDay();
                    var weekdayText = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];

                    var filteredBlogData = filterBlogData(blogData, pastDate);

                    if (filteredBlogData.length <= 0) {
                        break;
                    }

                    var formattedDate = year + "年" + (month < 10 ? "0" + month : month) + "月" + (day < 10 ? "0" + day : day) + "日";
                    var formattedDateWeekday = formattedDate + " " + weekdayText[weekday];

                    var blogList = document.getElementById("blogList");
                    var heading = document.createElement("h3");
                    heading.innerHTML = formattedDateWeekday;
                    blogList.appendChild(heading);

                    var ul = document.createElement("ul");

                    var randomNumber = 0;
                    var hash = CryptoJS.SHA256(formattedDate).toString();
                    for (var j = 0; j < 36; j += 9) {
                        randomNumber += parseInt(hash.slice(j, j + 8), 16) % (filteredBlogData.length + j);
                    }
                    randomNumber = randomNumber % filteredBlogData.length;

                    var blog = filteredBlogData[randomNumber];
                    var li = document.createElement("li");
                    var link = document.createElement("a");
                    link.style.color = "#3555BC";
                    link.href = blog.url;
                    link.innerHTML = blog.name;
                    li.appendChild(link);
                    ul.appendChild(li);
                    blogList.appendChild(ul);
                }
            }

                generateBlogList();
            </script>
        </div>

    </main>

    <div class="back-to-top" role="button" aria-label="返回顶部">
        <i class="fa fa-arrow-up fa-lg"></i>
        <span>0%</span>
    </div>

    <noscript>
        <div class="noscript-warning">Theme NexT works best with JavaScript enabled</div>
    </noscript>

    <script size="150" alpha="0.6" zIndex="-1"
        src="https://cdnjs.cloudflare.com/ajax/libs/ribbon.js/1.0.2/ribbon.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.2.1/anime.min.js"
        integrity="sha256-XL2inqUJaslATFnHdJOi9GfQ60on8Wx1C2H8DYiN1xY=" crossorigin="anonymous"></script>
    <!-- <script src="https://cdnjs.cloudflare.com/ajax/libs/next-theme-pjax/0.6.0/pjax.min.js"
        integrity="sha256-vxLn1tSKWD4dqbMRyv940UYw4sXgMtYcK6reefzZrao=" crossorigin="anonymous"></script> -->
    <script src="/js/comments.js"></script>
    <script src="/js/utils.js"></script>
    <script src="/js/motion.js"></script>
    <script src="/js/next-boot.js"></script>
    <!-- <script src="/js/pjax.js"></script> -->

    <script src="https://cdnjs.cloudflare.com/ajax/libs/hexo-generator-searchdb/1.4.1/search.js"
        integrity="sha256-1kfA5uHPf65M5cphT2dvymhkuyHPQp5A53EGZOnOLmc=" crossorigin="anonymous"></script>
    <script src="/js/third-party/search/local-search.js"></script>

    <script class="next-config" data-name="enableMath" type="application/json">true</script>
    <script class="next-config" data-name="mathjax"
        type="application/json">{"enable":true,"tags":"none","js":{"url":"https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/tex-mml-chtml.js","integrity":"sha256-MASABpB4tYktI2Oitl4t+78w/lyA+D7b/s9GEP0JOGI="}}</script>
    <script src="/js/third-party/math/mathjax.js"></script>


    <script src="https://unpkg.com/darkmode-js@1.5.7/lib/darkmode-js.min.js"></script>

    <script>
        var options = {
            bottom: '64px',
            right: 'unset',
            left: '32px',
            time: '0.5s',
            mixColor: 'transparent',
            backgroundColor: 'transparent',
            buttonColorDark: '#3F3F3F',
            buttonColorLight: '#fff',
            saveInCookies: true,
            label: '🌓',
            autoMatchOsTheme: true
        }
        const darkmode = new Darkmode(options);
        window.darkmode = darkmode;
        darkmode.showWidget();
    </script>

</body>

</html>
```

2. node_modules\hexo-generator-index\lib\generator.js

```js
'use strict';

const pagination = require('hexo-pagination');

module.exports = function(locals) {
  const config = this.config;
  const posts = locals.posts.sort(config.index_generator.order_by);

  // posts.data.sort((a, b) => (b.sticky || 0) - (a.sticky || 0));
  posts.data = posts.data.sort(function(a, b) {
      if(a.top && b.top) { // 两篇文章都有top，top大的在前
          if(a.top == b.top)
              return b.date - a.date; // 若top相等，最新的文章在前面
          else
              return b.top - a.top; // top大的在前面
      }
      else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面
          return -1;
      }
      else if(!a.top && b.top) {
          return 1;
      }
      else return b.date - a.date;  //都没有top标签，最新的文章在前面
  });
  
  const paginationDir = config.pagination_dir || 'page';
  const path = config.index_generator.path || '';

  return pagination(path, posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

3. 支持多行公式：https://blog.csdn.net/weixin_44634312/article/details/120197899

- 把hexo-renderer-marked换成了2018年的1.0.0版本，再修改escape
```shell
cd blog
npm uninstall hexo-renderer-marked
npm install hexo-renderer-marked@1.0.0
```

- 编辑node_modules/marked/lib/marked.js
```js
escape: /^\\([!"#$%&'()*+,\-./:;<=>?@\[\]\\^_`{|}~])/,
// 第539行，改成
escape: /^\\([!"#$&'()*+,\-./:;<=>?@\[\]^_`|~])/,

inline._escapes = /\\([!"#$%&'()*+,\-./:;<=>?@\[\]\\^_`{|}~])/g;
// 第564行，改成
inline._escapes = /\\([!"#$&'()*+,\-./:;<=>?@\[\]^_`|~])/g;
```

4. 主题添加可切换的暗黑模式：https://www.techgrow.cn/posts/abf4aee1.html
