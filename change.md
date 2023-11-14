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
- 可在项目中搜索<修改>二字，快速查找修改位置

4. 主题添加可切换的暗黑模式：https://www.techgrow.cn/posts/abf4aee1.html
