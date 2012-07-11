---
layout: default
title: conan.xxx
class: post
--- 

{{page.title}}
================================

it's rare these days that i get a chance to use html tables in any of the work i do. honestly, that's a very good thing. tables are prefect for tabular data, but anything even slightly beyond that is a nightmare. so a few weeks ago, when ask to build a parody porn site for [teamcoco](http://teamcoco.com), naturally my first thought was, "what is this 'porn' you speak of?" 

after seeing the mocks for [conan.xxx](http://conan.xxx), i was excited to see that the final design was very retro. i had immediate flashbacks to my early days building websites and the endless hours spent struggling to shove extremely simple layouts into tables. side track: i once spent two days pounding my head against a wall, trying to figure out why my entire site wouldn't render in [netscape 4](http://en.wikipedia.org/wiki/Netscape). after scouring the internet for hours (remember 56k modems, shit took longer), i finally [figured out](http://www.netmechanic.com/news/vol2/html_no4.htm) that i hadn't included the `</table>` tag.

conan.xxx was a perfect opportunity to dust off my long dormant table-based-layout skills and build an awesome parody porn site. here's just a sample, in all it's nested table glory:

{% highlight html %}
<body background="http://static.teamcococdn.com/microsite/conan.xxx/images/bg.gif" bgcolor="#debca7" text="white" link="#000000" vlink="#000000">
 <center>
  <table border="0" style="background:url(http://static.teamcococdn.com/microsite/conan.xxx/images/page.gif);" border="0" cellpadding="0" cellspacing="0" width="1052" height="1064">
   <tr>
    <td height="164" colspan="5"><h1 style="position:relative;width:100%;height:100%;margin:0"><img width="1006" height="194" src="http://static.teamcococdn.com/microsite/conan.xxx/images/logo.png" style="position:absolute;top:20px;left:30px" alt="Conan.xxx" width="" height=""></h1></td>
   </tr>
   <tr>
    <td height="359" width="32"><img alt="" src="http://static.teamcococdn.com/evergreen/blank.gif" width="100%" height="100%"></td>
    <td height="359" valign="top" width="640">
     <iframe id="video" width="640" height="359" src="http://teamcoco.com/embed/v/37278/skin=plain/autostart=true" frameborder="0" allowfullscreen></iframe>
    </td>  
    <td width="9"><img alt="" src="http://static.teamcococdn.com/evergreen/blank.gif" width="100%" height="100%"></td>   
    <td height="359" valign="top" width="300" align="center">      
     <table>
      <tr><td width="270" height="320" valign="top">
       <font face="Arial">
        <div id="copy">
         <h2>One Girl, Twenty Pencils</h2>
         <p>
          Meet Suzie. Don't let her calm exterior fool you, because she's even calmer then that! Watch her help us out with our sharpened pencil shortage.<br><br>
          Since we only have one video so far, the rest are videos of LaBamba doing whatever it is he does.
         </p>
        </div>
       </font>
      </td></tr>
     </table>
     <table style="margin-left: 20px" id="social">
      <tr>        
       <td><div class="g-plusone" data-size="medium"></div></td>
       <td><a href="https://twitter.com/share" data-url="http://conan.xxx" class="twitter-share-button" data-text="Conan.XXX - the third X is for 'XXX' - " data-via="teamcoco">Tweet</a></td>
       <td valign="top"><div class="fb-like" data-send="false" data-layout="button_count" data-width="40" data-show-faces="false"></div></td>        
      </tr>
     </table>           
    </td>
    <td height="359" width="60"><img alt="" src="http://static.teamcococdn.com/evergreen/blank.gif" width="100%" height="100%"></td>
   </tr>
    ...
{% endhighlight %}

my favorite bit of code:

{% highlight html %}
<td height="359" width="32"><img alt="" src="http://static.teamcococdn.com/evergreen/blank.gif" width="100%" height="100%"></td> 
{% endhighlight %}

transparent 1x1px GIFs, oh how i miss you.

you'll notice that i actually cheated quite a bit. there are some `style` attributes and some absolute positioning hacks. but i figured the page has a facebook like button, so any sense of pure 1999 era development would be gone anyway.

conan.xxx was a really fun project and is one of the reason i love working for [@teamcoco](http://twitter.com/teamcoco); every day brings something new and completely unexpected. it was also amazing to be reminded just how much web development has evolved since i started building website many years ago.

PS: if you haven't already, watch the videos on [conan.xxx](http://conan.xxx)... they're truly hilarious. props to [@Andy_Richter](https://twitter.com/andy_richter), [@dubouchet](https://twitter.com/dubouchet) and the other ['Conan' writers](https://twitter.com/#!/TeamCoco/conan-writers).

