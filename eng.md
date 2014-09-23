# CSS performance revisited: selectors, bloat and expensive styles 

*What is fast CSS? Where are the bottlenecks? Are the rules of slow and fast 
selectors even valid anymore? Are the properties we use more important than the 
selectors? I felt it was time to revisit some of these questions.*

In the broad scheme of things, CSS optimisation is certainly low down the
priority order when trying to speed websites/web applications up. There are so
many other optimisations that provide easier and greater gains. However, if tiny
improvements are made to all areas of a website, including CSS, combined they
will make a more substantial difference; the user will always benefit.

Whenever exchanging theories about the relative ‘speed’ of CSS, other developers
often reference [Steve Souders][1] work on CSS selectors from 2009. It’s used to
validate claims such as ‘attribute selectors are slow’ or ‘pseudo selectors are
slow’.

For at least the last couple of years, I’ve felt these kinds of things just
weren’t worth worrying about. The soundbite I have been wheeling out for years
is:

> With CSS, architecture is outside the braces; performance is inside

> [Ben Frain][2]

But besides referencing [Nicole Sullivan’s later post on Performance
Calendar][3] to back up my assumptions that the selectors used don’t really
matter, I had never actually tested the theory; a shortfall in my talent and a
less than perfect analytical mind prevented me from even attempting it.

Nothings changed with my mind, but these days I feel happier to open myself to
ridicule by attempting this – if only to get someone with more
knowledge/evidence to provide further data. So I decided to create some
primitive tests.

## Testing selector speed

Steve Souders’ aforementioned tests use JavaScript’s `new Date()`. However,
nowadays, modern browsers (iOS/Safari being notable exceptions) support the
Navigation Timing API which gives us a more accurate measure we can use. I’ll be
implementing it like this:

    <script type="text/javascript">
        ;(function TimeThisMother() {
            window.onload = function(){
                setTimeout(function(){
                var t = performance.timing;
                    alert("Speed of selection is: " + (t.loadEventEnd - t.responseEnd) + " milliseconds");
                }, 0);
            };
        })();
    </script>

This lets us limit the timing between the point all assets have been received
(`responseEnd`) and the point the page is rendered (`loadEventEnd`).

So, I setup a very simple test. 20 different pages, all with an identical,
enormous DOM, made up of 1000 identical chunks of this markup:

    <div class="tagDiv wrap1">
      <div class="tagDiv layer1" data-div="layer1">
        <div class="tagDiv layer2">
          <ul class="tagUl">
            <li class="tagLi"><b class="tagB"><a href="/" class="tagA link" data-select="link">Select</a></b></li>
          </ul>
        </div>
      </div>
    </div>

Each page differed only in the rule applied to select the inner most node within
the blocks. 20 different selection methods were tested to colour the inner most
nodes red:

1. [Data attribute][4]
2. [Data attribute (qualified)][5]
3. [Data attribute (unqualified but with value)][6]
4. [Data attribute (qualified with value)][7]
5. [Multiple data attributes (qualified with values)][8]
6. [Solo pseudo selector (e.g. :after)][9]
7. [Combined classes (e.g. class1.class2)][10]
8. [Multiple classes][11]
9. [Multiple classes with child selector][12]
10. [Partial attribute matching (e.g. [class^=“wrap”])][13]
11. [nth-child selector][14]
12. [nth-child selector followed by another nth-child selector][15]
13. [Insanity selection (all selections qualified, every class used e.g. div.wrapper > div.tagDiv > div.tagDiv.layer2 > ul.tagUL > li.tagLi > b.tagB > a.TagA.link)][16]
14. [Slight insanity selection (e.g. .tagLi .tagB a.TagA.link)][17]
15. [Universal selector][18]
16. [Element single][19]
17. [Element double][20]
18. [Element treble][21]
19. [Element treble with pseudo][22]
20. [Single class][23]

The test was run 5 times on each browser and the result averaged across the 5
results. Modern browsers were tested:

* Chrome 34.0.1838.2 dev
* Firefox 29.0a2 Aurora
* Opera 19.0.1326.63
* Internet Explorer 9.0.8112.16421
* Android 4.2 (7" tablet)

A prior version of Internet Explorer (rather than the latest IE) was used to
shed some light on how a popular browser behaved that doesn’t get the same
rolling frequent updates of the other browsers.

Want to try the same tests out for yourself? Go and grab the files from this
GitHub link: [https://github.com/benfrain/css-performance-tests][24]. Just open
each page in your browser of choice (remember the browser must support the
Network Timing API to alert a response). Also be aware that when I performed the
test I discarded the first couple of results as they tended to be unusually high
in some browsers.

When considering the results, I don’t think that one browser compared with
another really tells us much. That is not the purpose of the tests. The purpose
is purely to try and evaluate the comparative difference in selection speed
between the different selectors employed. Therefore, when looking at the table,
it makes more sense to look down the columns than across the rows.

Here are the results. All times in milliseconds:

<table>
<tr>
<td>Test</td>
<td>Chrome 34</td>
<td>Firefox 29</td>
<td>Opera 19</td>
<td>IE9</td>
<td>Android 4</td>
</tr>
<tr>
<td>1</td>
<td>56.8</td>
<td>125.4</td>
<td>63.6</td>
<td>152.6</td>
<td>1455.2</td>
</tr>
<tr>
<td>2</td>
<td>55.4</td>
<td>128.4</td>
<td>61.4</td>
<td>141</td>
<td>1404.6</td>
</tr>
<tr>
<td>3</td>
<td>55</td>
<td>125.6</td>
<td>61.8</td>
<td>152.4</td>
<td>1363.4</td>
</tr>
<tr>
<td>4</td>
<td>54.8</td>
<td>129</td>
<td>63.2</td>
<td>147.4</td>
<td>1421.2</td>
</tr>
<tr>
<td>5</td>
<td>55.4</td>
<td>124.4</td>
<td>63.2</td>
<td>147.4</td>
<td>1411.2</td>
</tr>
<tr>
<td>6</td>
<td>60.6</td>
<td>138</td>
<td>58.4</td>
<td>162</td>
<td>1500.4</td>
</tr>
<tr>
<td>7</td>
<td>51.2</td>
<td>126.6</td>
<td>56.8</td>
<td>147.8</td>
<td>1453.8</td>
</tr>
<tr>
<td>8</td>
<td>48.8</td>
<td>127.4</td>
<td>56.2</td>
<td>150.2</td>
<td>1398.8</td>
</tr>
<tr>
<td>9</td>
<td>48.8</td>
<td>127.4</td>
<td>55.8</td>
<td>154.6</td>
<td>1348.4</td>
</tr>
<tr>
<td>10</td>
<td>52.2</td>
<td>129.4</td>
<td>58</td>
<td>172</td>
<td>1420.2</td>
</tr>
<tr>
<td>11</td>
<td>49</td>
<td>127.4</td>
<td>56.6</td>
<td>148.4</td>
<td>1352</td>
</tr>
<tr>
<td>12</td>
<td>50.6</td>
<td>127.2</td>
<td>58.4</td>
<td>146.2</td>
<td>1377.6</td>
</tr>
<tr>
<td>13</td>
<td>64.6</td>
<td>129.2</td>
<td>72.4</td>
<td>152.8</td>
<td>1461.2</td>
</tr>
<tr>
<td>14</td>
<td>50.2</td>
<td>129.8</td>
<td>54.8</td>
<td>154.6</td>
<td>1381.2</td>
</tr>
<tr>
<td>15</td>
<td>50</td>
<td>126.2</td>
<td>56.8</td>
<td>154.8</td>
<td>1351.6</td>
</tr>
<tr>
<td>16</td>
<td>49.2</td>
<td>127.6</td>
<td>56</td>
<td>149.2</td>
<td>1379.2</td>
</tr>
<tr>
<td>17</td>
<td>50.4</td>
<td>132.4</td>
<td>55</td>
<td>157.6</td>
<td>1386</td>
</tr>
<tr>
<td>18</td>
<td>49.2</td>
<td>128.8</td>
<td>58.6</td>
<td>154.2</td>
<td>1380.6</td>
</tr>
<tr>
<td>19</td>
<td>48.6</td>
<td>132.4</td>
<td>54.8</td>
<td>148.4</td>
<td>1349.6</td>
</tr>
<tr>
<td>20</td>
<td>50.4</td>
<td>128</td>
<td>55</td>
<td>149.8</td>
<td>1393.8</td>
</tr>
<tr>
<td>Biggest Diff.</td>
<td>16</td>
<td>13.6</td>
<td>17.6</td>
<td>31</td>
<td>152</td>
</tr>
<tr>
<td>Slowest</td>
<td>13</td>
<td>6</td>
<td>13</td>
<td>10</td>
<td>6</td>
</tr>
</table>
 
## The difference between fastest and slowest selector

The Biggest Diff. column shows the difference in milliseconds between the
fastest and slowest selector. Of the desktop browsers, IE9 stands out as having
the biggest difference between fastest and slowest selectors at 31ms. The others
are all around half of that figure. However, interestingly there was no
consensus on what the slowest selector was.

## The slowest selector

I was interested to note that the slowest selector type differed from browser to
browser. Both Opera and Chrome found the ‘insanity’ selector (test 13) the
hardest to match (the similarity Opera and Chrome here perhaps not surprising
given their shared blink engine), while Firefox struggled with a single pseudo
selector ([test 6][25]), as did the Android 4.2 device (a Tesco hudl 7" tablet).
Internet Explorer 9’s achilles heel was the partial attribute selector ([test
10][26]).

## Good CSS architecture practices

One thing we can be clear on is that using a flat hierarchy of class based
selectors not only produces more modular and less specific code making it more
modular and re-usable, it also provides selectors that are as fast as any others
(yes, ID selectors would probably be faster but I for one don’t fancy building a
large code base up relying on ID selectors).

### What does this mean?

For me, it has confirmed my believe that it is absolute folly to worry about the
type of selector used. Second guessing a selector engine is pointless as the
manner selector engines work through selectors clearly differs. Further more,
the difference between fastest and slowest selectors isn’t massive, even on a
ludicrous DOM size like this. As we say in the North of England, ‘There are
bigger fish to fry’.

Since originally writing this post, Benjamin Poulain, a WebKit Engineer got in
touch to point out his concerns with the methodology used. His comments were
very interesting and with his permission, some of the information is quoted
below:

“By choosing to measure performance through the loading, you are measuring 
plenty of much much bigger things than CSS, CSS Performance is only a small part 
of loading a page:

If I take the time profile of `[class^="wrap"]` for example (taken on an old
WebKit so that it is somewhat similar to Chrome), I see:

~10% of the time is spent in the rasterizer. ~21% of the time is spent on the 
first layout. ~48% of the time is spent in the parser and DOM tree creation ~8% 
is spent on style resolution ~5% is spent on collecting the style – this is what 
we should be testing and what should take most of the time. (The remaining time 
is spread over many many little functions).

With the test above, let say we have a baseline of 100 ms with the fastest
selector. Of that, 5 ms would be spent collecting style. If a second selector is
3 times slower, that would appear as 110ms in total. The test should report a
300% difference but instead it only shows 10%.”

At this point, I responded that whilst I understood what Benjamin was pointing
our, my test was only supposed to illustrate that the same page, with all other
things being equal, renders largely the same irregardless of the selector used.
Benjamin took the time to reply with further detail:

“I completely agree it is useless to optimize selectors upfront, but for 
completely different reasons:

It is practically impossible to predict the final performance impact of a given
selector by just examining the selectors. In the engine, selectors are
reordered, split, collected and compiled. To know the final performance of a
given selectors, you would have to know in which bucket the selector was
collected, how it is compiled, and finally what does the DOM tree looks like.

All of that is very different between the various engines, making the whole
process even less predictable.

The second argument I have against web developers optimizing selectors is that
they will likely make things worse. The amount of misinformation about selectors
is larger than correct cross-browser information. The chance of someone doing
the right thing is pretty low.

In practice, people discover performance problems with CSS and start removing
rules one by one until the problem go away. I think that is the right way to go
about this, it is easy and will lead to correct outcome.”

## Cause and effect

If the number of DOM elements on the page is halved, as you would expect, the
speed to complete any of the test drops commensurately. But getting rid of the
DOM isn’t always a possibility. This made me wonder what difference the amount
of unused styles in the CSS would have on the results.

## What difference to selection speed does a whole lot of unused styles make?

[Another test][27]: I grabbed a big fat style sheet from fiat.co.uk. It was 
about 3000 lines of CSS. All these irrelevant styles were inserted before a 
final rule that would select our inner `a.link` node and make it red. I did the 
same averaging of the results across 5 runs on each browser.

I [then cut half those rules out and repeated the test][28] to give a
comparison. Here are the results:

<table>
<tr>
<td>Test</td>
<td>Chrome 34</td>
<td>Firefox 29</td>
<td>Opera 19</td>
<td>IE9</td>
<td>Android 4</td>
</tr>
<tr>
<td>Full bloat</td>
<td>64.4</td>
<td>237.6</td>
<td>74.2</td>
<td>436.8</td>
<td>1714.6</td>
</tr>
<tr>
<td>Half bloat</td>
<td>51.6</td>
<td>142.8</td>
<td>65.4</td>
<td>358.6</td>
<td>1412.4</td>
</tr>
</table>

## Style diet

This provides some interesting figures. For example, Firefox was 1.7X slower to
complete this test than it was with its slowest selector test (test 6). Android
4.3 was 1.2X slower than its slowest selector test (test 6). Internet Explorer
was a whopping 2.5X slower than it’s slowest selector!

You can see that things dropped down considerably for Firefox when half of the
styles were removed (approx 1500 lines). The Android device came down to around
the speed of its slowest selector at that point too.

### Removing unused styles

Does this kind of horror scenario sound familiar to you? Enormous CSS files with
all manner of selectors (often with selectors in that don’t even work), lumps of
ever more specific selectors 7 or more levels deep, non-applicable prefix’s, IDs
all over the shop and file sizes of 50–80KB (sometimes more).

If you are working on a code base that has a big fat CSS file like this, that
no-one is quite sure what all the styles are actually for – look there for your
CSS optimisations before the selectors being employed.

Tackling this first seems to make more sense than being picky over the selectors
used. It will have double the impact; less code for the user to download but
also less for the UA to parse – a speed bump all around.

Then again, that won’t help with the actual performance of your CSS.

## Performance inside the brackets

The [final test][29] I ran was to hit the page with a bunch of ‘expensive’
properties and values.

    .link {
        background-color: red;
        border-radius: 5px;
        padding: 3px;
        box-shadow: 0 5px 5px #000;
        -webkit-transform: rotate(10deg);
        -moz-transform: rotate(10deg);
        -ms-transform: rotate(10deg);
        transform: rotate(10deg);
        display: block;
    }

With that rule applied, here are the results:

<table>
<tr>
<td>Test</td>
<td>Chrome 34</td>
<td>Firefox 29</td>
<td>Opera 19</td>
<td>IE9</td>
<td>Android 4</td>
</tr>
<tr>
<td>Expensive Styles</td>
<td>65.2</td>
<td>151.4</td>
<td>65.2</td>
<td>259.2</td>
<td>1923</td>
</tr>
</table>

Here all browsers are at least up with their slowest selector speed (IE was 1.5X
slower than its slowest selector test (10) and the Android device was 1.3X
slower than the slowest selector test (test 6)) but that’s not even the full
picture. Try and scroll! Repaint on these kind of styles will make your computer
cry.

The properties we stick inside the braces are what really taxes a system. It
stands to reason that scrolling a page that requires endless expensive re-paints
and layout changes is going to put a strain on the device. Nice HiDPI screen? It
will be even worse as the CPU/GPU strains to get everything re-painted to screen
in under 16ms.

With the expensive styles test, on the 15" Retina MacBook Pro I tested on, the
paint time shown in continuous paint mode in Chrome never dropped below 280ms
(and remember, we are aiming for sub–16ms). To put that in perspective for you,
the first selector test page, never went above 2.5ms. That wasn’t a typo. Those
properties created a 112X increase in paint time. Holy ’effing expensive
properties Batman! Indeed Robin. Indeed.

## What properties are expensive?

An ‘expensive’ property/value pairing is one we can be pretty confident will
make the browser struggle with when it has to repaint the screen (e.g. on
scroll).

How can we know what will be an ‘expensive’ style? Thankfully, we can apply
common sense to this and get a pretty good idea what is going to tax the
browser. Anything that requires a browser to manipulate/calculate before
painting to the page will be more costly. For example, box-shadows, border-
radius, transparency (as the browser has to calculate what is shown below),
transforms and performance killers like CSS filters – if performance is your
priority, anything like that is your worst enemy.

Juriy “kangax” Zaytsev did [a fantastic blog post also covering CSS
performance][30] back in 2012. He was using the various developer tools to
measure performance. He did a particularly good job of showing the difference
that various properties had on performance. If this kind of thing interests you
then that post is well worth your time.

## Conclusion

These are my takeaways from this little episode: - sweating over the selectors
used in modern browsers is futile; most selection methods are now so fast it’s
really not worth spending much time over. Furthermore, there is disparity across
browsers of what the slowest selectors are anyway. Look here last to speed up
your CSS. - excessive unused styles are likely to cost more, performance wise,
than any selectors you chose so look to tidy up there second. 3000 lines that
are unused or surplus on a page are not even that uncommon. While it’s common to
bunch all the styles up into a great big single `styles.css`, if different areas
of your site/web application can have different (additional) stylesheets added
(dependency graph style), that may be the better option. If your CSS has been
added to by a number of different authors over time, look to tools like
[UnCSS][31] to automate the removal of styles – doing that process by hand is no
fun! - the battle for high performing CSS will not be won in the selectors used,
it will be won with the judicious use of property and values - getting something
painted to screen fast is obviously important but so is how a page feels when
the user interacts with it. Look for expensive property and value pairs first
(Chrome continuous repaint mode is your friend here), they are likely to provide
the biggest gains.

[1]: http://stevesouders.com/
[2]: http://benfrain.com/
[3]: http://calendar.perfplanet.com/2011/css-selector-performance-has-changed-for-the-better/
[4]: http://benfrain.com/selector-test/01.html
[5]: http://benfrain.com/selector-test/02.html
[6]: http://benfrain.com/selector-test/03.html
[7]: http://benfrain.com/selector-test/04.html
[8]: http://benfrain.com/selector-test/05.html
[9]: http://benfrain.com/selector-test/06.html
[10]: http://benfrain.com/selector-test/07.html
[11]: http://benfrain.com/selector-test/08.html
[12]: http://benfrain.com/selector-test/09.html
[13]: http://benfrain.com/selector-test/10.html
[14]: http://benfrain.com/selector-test/11.html
[15]: http://benfrain.com/selector-test/12.html
[16]: http://benfrain.com/selector-test/13.html
[17]: http://benfrain.com/selector-test/14.html
[18]: http://benfrain.com/selector-test/15.html
[19]: http://benfrain.com/selector-test/16.html
[20]: http://benfrain.com/selector-test/17.html
[21]: http://benfrain.com/selector-test/18.html
[22]: http://benfrain.com/selector-test/19.html
[23]: http://benfrain.com/selector-test/20.html
[24]: https://github.com/benfrain/css-performance-tests
[25]: http://benfrain.com/selector-test/06.html
[26]: http://benfrain.com/selector-test/10.html
[27]: http://benfrain.com/selector-test/2-01.html
[28]: http://benfrain.com/selector-test/2-02.html
[29]: http://benfrain.com/selector-test/3-01.html
[30]: http://perfectionkills.com/profiling-css-for-fun-and-profit-optimization-notes/
[31]: https://github.com/giakki/uncss