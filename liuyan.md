---
layout: page
title: 留言板「MESSAGE」 
---

<img src="/images/avatar.jpg" alt="huanying"/>


<p><h4>有什么话要对我说吗？</h4>     
<P><h4>这里是你畅所欲言的地方，可以咨询，</h4>
<p><h4>可以交流，可以感叹，可以发飙，但不可以订外卖 、、、</h4>   
<p>
<div>  
           
 
<object width="330" height="180" data="http://music.163.com/style/swf/widget.swf?sid=441877316&type=0&auto=1&width=310&height=430" type="application/x-shockwave-flash"></object>  
          
  </div> 
-->


<div id="QPlayer" class="QPlayer">
<div id="pContent">
	<div id="player">
<span class="cover"></span>
<div class="ctrl">
<div class="musicTag marquee">
<strong>Title</strong>
<span> - </span>
<span class="artist">Artist</span>
</div>
<div class="progress">
<div class="timer left">0:00</div>
<div class="contr">
<div class="rewind icon"></div>
<div class="playback icon"></div>
<div class="fastforward icon"></div>
</div>
<div class="right">
<div class="liebiao icon"></div>
</div>
</div>
</div>
</div>
	<div class="ssBtn">
	        <div class="adf"></div>
    </div>
</div>
<ol id="playlist"></ol>
</div>

<script src="/js/jquery.min.js"></script>
<script src="/js/jquery.marquee.min.js"></script>

<script>
	var	playlist = [
{title:"My All",artist:"浜崎あゆみ",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/%E6%B5%9C%E5%B4%8E%E3%81%82%E3%82%86%E3%81%BF%20-%20MY%20ALL.mp3",cover:"http://p4.music.126.net/7VJn16zrictuj5kdfW1qHA==/3264450024433083.jpg?param=106x106",},
{title:"Try Everything",artist:"Shakira",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/Shakira%20-%20Try%20Everything.mp3",cover:"http://p4.music.126.net/KLw_TLTRUe9pClPv4vlEtQ==/936783906865219.jpg?param=106x106",},
{title:"Victory",artist:"Two Steps From Hell",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/Two%20Steps%20From%20Hell%20-%20Victory.mp3",cover:"http://p4.music.126.net/YXY1vPG5rtdV7w_cWDnNWw==/884007348732141.jpg?param=106x106",},
{title:"Monody",artist:"TheFatRat,Laura Brehm",mp3:"http://omjh2j5h3.bkt.clouddn.com/TheFatRat,Laura%20Brehm%20-%20Monody.mp3",cover:"http://p3.music.126.net/1odRfg3HXWmYw02EMXKRKQ==/116548232557498.jpg?param=106x106",},
{title:"Luv Letter",artist:"dj okawari ",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/Luv%20Letter.mp3",cover:"http://p4.music.126.net/F2fqWwTTT2DAOKPQKQ-G0A==/5892282813545901.jpg?param=106x106",},
{title:"Born this way",artist:"lady gaga ",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/Born%20this%20way.mp3",cover:"http://p4.music.126.net/G2nCsXpMc81lcUY-pOHr9Q==/2528876745541310.jpg?param=106x106",},
{title:"The Edge of Glory",artist:"Lady Gaga",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/The%20Edge%20of%20Glory.mp3",cover:"http://p3.music.126.net/iYG3tZ2xSKrzf65BaDtEJQ==/7929677860524772.jpg?param=106x106",},
{title:"Beautiful",artist:"Eminem ",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/Beautiful.mp3",cover:"http://p4.music.126.net/F2fqWwTTT2DAOKPQKQ-G0A==/5892282813545901.jpg?param=106x106",},
{title:"Hall of Fame",artist:"the script/will.i.am",mp3:"http://omjh2j5h3.bkt.clouddn.com/music/Hall%20of%20Fame.mp3",cover:"http://p4.music.126.net/d5ryd0uwq29KWk3bRZ1wsA==/45079976751142.jpg?param=106x106",},
{title:"I Saw Him",artist:"I Saw Him",mp3:"http://m10.music.126.net/20171027120549/329b3b62e2cbcff379f4ff56bf20f650/ymusic/6b21/1e61/4219/705cc3e8859535e52ea3ed5134dc1ef4.mp3",cover:"http://p1.music.126.net/PaCWRxXASgHp2yHl6E6w-g==/3262251001467554.jpg??param=106x106",},
];
  var isRotate = true;
  var autoplay = true;
</script>
<script src="/js/player.js"></script>
<script>

function bgChange(){
	var lis= $('.lib');
	for(var i=0; i<lis.length; i+=2)
	lis[i].style.background = 'rgba(246, 246, 246, 0.5)';
}
window.onload = bgChange;
</script>

<meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1" />
	<title></title>
	<link rel="stylesheet" href="/css/player.css">



<script>
myVid=document.getElementById("audio1");

function setHalfVolume()
  { 
  myVid.volume=0.2;
  } 

</script> 


<!-- 多说评论框 start 
	<div class="ds-thread" data-thread-key="/liuyan/" data-title="留言板" data-url="http://roboutkang/liuyan/"></div>
<!-- 多说评论框 end 
<!-- 多说公共JS代码 start (一个网页只需插入一次) 
<script type="text/javascript">
var duoshuoQuery = {short_name:"robotkang"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] 
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
	</script>
<!-- 多说公共JS代码 end -->


<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Valine - A simple comment system based on Leancloud.</title>
    <!--Leancloud 操作库:-->
    <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
    <!--Valine 的核心代码库:-->
    <script src="/Valine-1.1.4/dist/Valine.min.js"></script>
</head>
<body>
    <div class="comment"></div>
    <script>
       new Valine({
       // AV 对象来自上面引入av-min.js(老司机们不要开车➳♡゛扎心了老铁)
      		 av: AV, 
            el: '.comment', // 
            app_id: 'CWa3VWGExyX5qBwL6BQ7mwHv-gzGzoHsz', // 这里填写上面得到的APP ID
            app_key: 'KxtOwoN1GTvwt1daWglcosv8', // 这里填写上面得到的APP KEY
            placeholder: 'ヾﾉ≧∀≦)o来啊，快活啊!' // [v1.0.7 new]留言框占位提示文字
       });
</script>
</body>
</html>


<p>
<a href="/fangke/" style="color:#708090"> <h5>Recent Visitors</h5></a>  
</p>



