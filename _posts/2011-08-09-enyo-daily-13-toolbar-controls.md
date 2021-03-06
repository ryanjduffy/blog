---
layout: post
title: 'Enyo Daily #13 - Toolbar Controls'
date: 2011-08-09 09:22 -0700
---

<p><p>One of the reasons I enjoy working with webOS is its foundation of standard web technologies.  In practice, this translates to the ability to customize the behavior and display of the framework in ways its creators didn't foresee.  One such mechanism is CSS -- parodoxically my most and least favorite.  I love that I can throw a -webkit-border-radius on a &lt;div&gt; rather than opening up a graphics program and the <a href="http://blog.technisode.com/post/8055838446/enyo-daily-4-box-model" target="_blank">flexible box model is amazing</a> (coming from someone who lived by nested tables back in the day).</p>
<p>HP has already covered most of the built-in Toolbar controls in their <a href="https://developer.palm.com/content/api/dev-guide/enyo/toolbars.html" target="_blank">Enyo Developer Guide</a> but there's nothing preventing you from dropping any control you want in a Toolbar.  I'll cover a couple examples and show how reusing existing enyo CSS classes and tweaking others can make other controls look at home in the Toolbar.</p>
[[MORE]]
<p>I'll start with covering a couple controls the dev guide leaves out:  ToolInput, ToolInputBox, and ToolSearchInput.  In each case, the controls are identical to their non-Toolbar counterparts with the exception of the CSS classes applied.  In fact, each inherits directly from their respective "normal" control.  I mention them here mainly to illustrate a template for how you might ready other controls (Enyo or custom) to be displayed in a Toolbar.<br><br>To illustrate, I create a new ToolSlider and ToolCustomListSelector control.  ToolSlider inherits from Slider and adds the classes from the ToolInput control.  I also added an extras-tool-slider class to reduce the margins to make it fit better. </p>
<blockquote>
<p>enyo.kind({<br>  name:"ToolSlider",<br>  kind:"Slider",<br>  className:"extras-tool-slider enyo-input enyo-tool-input"<br>});</p>
</blockquote>
<p>ToolCustomListSelector inherits from CustomListSelector (surprising, I know) and adds the classes from ToolButton.  Again, I added an additional class, extras-tool-listselector, to manage the height of the control.</p>
<blockquote>
<p>enyo.kind({<br>  name:"ToolCustomListSelector",<br>  kind:"CustomListSelector",<br>  className:"extras-tool-listselector enyo-tool-button-client enyo-tool-button-captioned"<br>});</p>
</blockquote>
<p>Here's everything -- all the built-in controls plus the 2 custom controls above -- in action.</p>
<p>CSS</p>
<blockquote>
<p>.extras-tool-listselector .enyo-listselector-arrow {<br>    min-height:20px;<br>}<br><br>.extras-tool-slider .enyo-slider-progress {<br>    margin:10px 5px;<br>}</p>
</blockquote>
<p>JavaScript</p>
<blockquote>
<p>var _Example = {<br>    name:"com.technisode.example.App",<br>    kind:"Control",<br>    components:[<br>        {kind:"Toolbar", name:"tb1", pack:"justify", name:"toolbar", components:[<br>            {kind:"ToolButton", caption:"Button"},<br>            {kind:"ToolInput", hint:"important stuff"},<br>            {kind:"ToolSearchInput"},<br>            {kind:"RadioToolButtonGroup", components:[<br>                  {label:"First"},<br>                  {label:"Second"},<br>                  {label:"Third"},<br>            ]},<br>            {kind:"ToolButtonGroup", components:[<br>                {kind:"GroupedToolButton", caption:"Button"},<br>                {kind:"GroupedToolButton", caption:"Button"},<br>                {kind:"GroupedToolButton", caption:"Button"},<br>            ]},<br>        ]},<br>        {kind:"Toolbar", name:"tb2", pack:"justify", components:[<br>            {kind:"ToolInputBox", width:"300px", components:[<br>                 {kind:"Input", styled:false, flex:1},<br>                 {kind: "ToolCustomListSelector", value: 3, items: [<br>                     {caption: "Google", value: 1},<br>                     {caption: "Bing", value: 2},<br>                     {caption: "Alta Vista", value: 3},<br>                 ]},<br>            ]},<br>            {kind: "ToolCustomListSelector", value: 2, items: [<br>                {caption: "One", value: 1},<br>                {caption: "Two", value: 2},<br>                {caption: "Three", value: 3},<br>            ]},<br>            {kind:"ToolSlider", maximum:10, minimum:0, value:5, width:"200px"},<br>        ]}<br>    ]<br>}<br><br>enyo.kind({name:"ToolCustomListSelector", kind:"CustomListSelector", className:"extras-tool-listselector enyo-tool-button-client enyo-tool-button-captioned"})<br>enyo.kind({name:"ToolSlider", kind:"Slider", className:"extras-tool-slider enyo-input enyo-tool-input"})<br>enyo.kind(_Example);</p>
</blockquote></p>