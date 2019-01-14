> ​	每个公司业务不一样，此解决方案仅做参考 。

[Git demo地址](https://github.com/youhunwl/i18n)

## 更新(2018-07-17)

默认当前系统设置语言;
考虑到一些表单输入框`placeholder`也有中文，加`lang-input`翻译。

## 前言

其实现在开发者解决多语言普遍三种解决方案：

第一个是为每个页面提供每种语言的相关页面。
第二种是把内容从表现形式中分离出来，做不同语言的内容文件。
第三种是动态翻译页面内容。第三种很少见，而且机器翻译技术还很难达到人们的预期。

其实第二种相对来说简单一点，那么开搞。

## 实现

### 思考

* 翻译公司给的有的excel有的是json文件，咱们就统一请求json文件吧;
* html中给标签加个`lang`属性，到时候页面加载时遍历所有这些有`lang`属性的标签去实现切换语言;
* js里的文字用方法实现转换语言;
* 把用户选择的语言存到cookie里吧，嗯！拿个小本本记下来;
* 做个缓存，请求过的语言文件就不再请求了;
* 暂时就这些吧...

### demo

![](https://ws3.sinaimg.cn/large/005BYqpgly1freeqgvy4aj30xr0fcwhd.jpg)

**文件目录**

![](https://ws3.sinaimg.cn/large/005BYqpgly1fredi3sjrkj307405dwee.jpg)

**index.html**

```html
<!DOCTYPE html>
<html>
<meta charset="utf-8">  
<title>translation test</title>  
<script src="https://cdn.bootcss.com/jquery/1.11.0/jquery.min.js"></script>
<script src="js/script.js"></script>  
<script src="js/index.js"></script>  
</head>  
  
<body>  
    <div>  
        <a href="#" id="enBtn">English</a>  
        <a href="#" id="zhBtn">简体中文</a>  
    </div>  
    <div><a lang>click here:</a></div>  
    <div><input type="button" value="apply" lang id="applyBtn"></div> 
</body>  
  
</html>  

```

**script.js**

```javascript
var dict = {};
var systemLang = navigator.language.toLowerCase().slice(0,2);
$(function () {
  registerWords();
  if(getCookieVal("lang")=="en"){
    setLanguage("en");
  }else if(getCookieVal("lang")=="zh"){
    setLanguage("zh");
  }else{
    setLanguage(systemLang);
  }
  
// 切换语言事件 根据自己业务而定
  $("#enBtn").bind("click", function () {
    setLanguage("en");
    //这里也可以写一些其他操作，比如加载语言对应的css
  });

  $("#zhBtn").bind("click", function () {
    setLanguage("zh");
  });

});

function setLanguage(lang) {
  setCookie("lang=" + lang + "; path=/;");
  translate(lang);
}

function getCookieVal(name) {
  var items = document.cookie.split(";");
  for (var i in items) {
    var cookie = $.trim(items[i]);
    var eqIdx = cookie.indexOf("=");
    var key = cookie.substring(0, eqIdx);
    if (name == $.trim(key)) {
      return $.trim(cookie.substring(eqIdx + 1));
    }
  }
  return null;
}

function setCookie(cookie) {
  var Days = 30; //此 cookie 将被保存 30 天
  var exp = new Date(); //new Date("December 31, 9998");
  exp.setTime(exp.getTime() + Days * 24 * 60 * 60 * 1000);
  document.cookie = cookie+ ";expires=" + exp.toGMTString();
}

function translate(lang) {
  if(sessionStorage.getItem(lang + "Data") != null){
    dict = JSON.parse(sessionStorage.getItem(lang + "Data"));
  }else{
    loadDict();
  }

  $("[lang]").each(function () {
    switch (this.tagName.toLowerCase()) {
      case "input":
        $(this).val(__tr($(this).attr("lang")));
        break;
      default:
        $(this).text(__tr($(this).attr("lang")));
    }
  });
  $("[lang-input]").each(function () {
    $(this).attr("placeholder", __tr($(this).attr("lang-input")));
  });
}

function __tr(src) {
  return (dict[src] || src);
}

function loadDict() {
  var lang = (getCookieVal("lang") || "en");
  $.ajax({
    async: false,
    type: "GET",
    url: "/lang/"+lang + ".json",
    success: function (msg) {
      dict = msg;
      sessionStorage.setItem(lang + 'Data', JSON.stringify(dict));
    }
  });

}

// 遍历所有lang属性的标签赋值
function registerWords() {
  $("[lang]").each(function () {
    if ($(this).attr("lang") === "") {
      switch (this.tagName.toLowerCase()) {
        case "input":
          $(this).attr("lang", $(this).val());
          break;
        default:
          $(this).attr("lang", $(this).text());
      }
    }
  });
   $("[lang-input]").each(function () {
    if ($(this).attr("lang-input") === "") {
      $(this).attr("lang-input", $(this).attr("placeholder"));
    }
  });
}

```

之前弄demo的时候，`registerWords`函数这里没有判断
但是我们的项目自己封装的路由去动态加载页面。每次进来都会重新赋值，这会导致问题。

因为他赋值的是当前元素的值，这个时候你`lang`的值就和语言包文件里的`key`对应不上了

### 使用方法

html中语言切换：给所有标签加上`lang`属性
js中语言切换：使用`__tr()`方法

可以直接把`script.js`作为一个插件使用放到项目中

## 总结

条条大路通罗马，根据自己的实际需求与业务场景去做即可。

有点仓促，有不足的还请各位指点。
