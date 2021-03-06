﻿
        <p>准确地讲，Python没有专门处理字节的数据类型。但由于<code>b&#39;str&#39;</code>可以表示字节，所以，字节数组＝二进制str。而在C语言中，我们可以很方便地用struct、union来处理字节，以及字节和int，float的转换。</p>
<p>在Python中，比方说要把一个32位无符号整数变成字节，也就是4个长度的<code>bytes</code>，你得配合位运算符这么写：</p>
<pre><code>&gt;&gt;&gt; n = 10240099
&gt;&gt;&gt; b1 = (n &amp; 0xff000000) &gt;&gt; 24
&gt;&gt;&gt; b2 = (n &amp; 0xff0000) &gt;&gt; 16
&gt;&gt;&gt; b3 = (n &amp; 0xff00) &gt;&gt; 8
&gt;&gt;&gt; b4 = n &amp; 0xff
&gt;&gt;&gt; bs = bytes([b1, b2, b3, b4])
&gt;&gt;&gt; bs
b&#39;\x00\x9c@c&#39;
</code></pre><p>非常麻烦。如果换成浮点数就无能为力了。</p>
<p>好在Python提供了一个<code>struct</code>模块来解决<code>bytes</code>和其他二进制数据类型的转换。</p>
<p><code>struct</code>的<code>pack</code>函数把任意数据类型变成<code>bytes</code>：</p>
<pre><code>&gt;&gt;&gt; import struct
&gt;&gt;&gt; struct.pack(&#39;&gt;I&#39;, 10240099)
b&#39;\x00\x9c@c&#39;
</code></pre><p><code>pack</code>的第一个参数是处理指令，<code>&#39;&gt;I&#39;</code>的意思是：</p>
<p><code>&gt;</code>表示字节顺序是big-endian，也就是网络序，<code>I</code>表示4字节无符号整数。</p>
<p>后面的参数个数要和处理指令一致。</p>
<p><code>unpack</code>把<code>bytes</code>变成相应的数据类型：</p>
<pre><code>&gt;&gt;&gt; struct.unpack(&#39;&gt;IH&#39;, b&#39;\xf0\xf0\xf0\xf0\x80\x80&#39;)
(4042322160, 32896)
</code></pre><p>根据<code>&gt;IH</code>的说明，后面的<code>bytes</code>依次变为<code>I</code>：4字节无符号整数和<code>H</code>：2字节无符号整数。</p>
<p>所以，尽管Python不适合编写底层操作字节流的代码，但在对性能要求不高的地方，利用<code>struct</code>就方便多了。</p>
<p><code>struct</code>模块定义的数据类型可以参考Python官方文档：</p>
<p><a href="https://docs.python.org/3/library/struct.html#format-characters">https://docs.python.org/3/library/struct.html#format-characters</a></p>
<p>Windows的位图文件（.bmp）是一种非常简单的文件格式，我们来用<code>struct</code>分析一下。</p>
<p>首先找一个bmp文件，没有的话用“画图”画一个。</p>
<p>读入前30个字节来分析：</p>
<pre><code>&gt;&gt;&gt; s = b&#39;\x42\x4d\x38\x8c\x0a\x00\x00\x00\x00\x00\x36\x00\x00\x00\x28\x00\x00\x00\x80\x02\x00\x00\x68\x01\x00\x00\x01\x00\x18\x00&#39;
</code></pre><p>BMP格式采用小端方式存储数据，文件头的结构按顺序如下：</p>
<p>两个字节：<code>&#39;BM&#39;</code>表示Windows位图，<code>&#39;BA&#39;</code>表示OS/2位图；
一个4字节整数：表示位图大小；
一个4字节整数：保留位，始终为0；
一个4字节整数：实际图像的偏移量；
一个4字节整数：Header的字节数；
一个4字节整数：图像宽度；
一个4字节整数：图像高度；
一个2字节整数：始终为1；
一个2字节整数：颜色数。</p>
<p>所以，组合起来用<code>unpack</code>读取：</p>
<pre><code>&gt;&gt;&gt; struct.unpack(&#39;&lt;ccIIIIIIHH&#39;, s)
(b&#39;B&#39;, b&#39;M&#39;, 691256, 0, 54, 40, 640, 360, 1, 24)
</code></pre><p>结果显示，<code>b&#39;B&#39;</code>、<code>b&#39;M&#39;</code>说明是Windows位图，位图大小为640x360，颜色数为24。</p>
<p>请编写一个<code>bmpinfo.py</code>，可以检查任意文件是否是位图文件，如果是，打印出图片大小和颜色数。</p>
<h3 id="-">参考源码</h3>
<p><a href="https://github.com/michaelliao/learn-python3/blob/master/samples/commonlib/check_bmp.py">check_bmp.py</a></p>

    