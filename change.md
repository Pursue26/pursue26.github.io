代码修改：
1. node_modules\hexo-generator-random\random.html

```js
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
    var year = currentDate.getFullYear();
    var month = currentDate.getMonth() + 1; // 月份从 0 开始，需要加 1
    var day = currentDate.getDate();
    // var seconds = currentDate.getSeconds();
    var formattedDate = `${year}-${month}-${day}`;

    var savedDate = JSON.parse(localStorage.getItem('savedHexoDate'));
    var savedN = JSON.parse(localStorage.getItem('savedHexoN'));

    var n = 0;
    if(savedDate && savedDate === formattedDate) {
        n = savedN;
    } else {
        n = Math.floor(Math.random() * filteredUrls.length);
        localStorage.setItem('savedHexoDate', JSON.stringify(formattedDate));
        localStorage.setItem('savedHexoN', JSON.stringify(n));
    }

    // console.log(n);
    location.href = urls[n % filteredUrls.length];
}
</script>
</body>
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