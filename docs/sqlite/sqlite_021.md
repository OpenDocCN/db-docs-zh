# 1\. 窗口函数介绍

> 原文：[`sqlite.com/windowfunctions.html`](https://sqlite.com/windowfunctions.html)

窗口函数是一种 SQL 函数，其输入值来自于 SELECT 语句结果集中的一个或多个行的“窗口”。

窗口函数通过存在**OVER**子句与标量函数和聚合函数区分开来。如果一个函数有 OVER 子句，则它是一个窗口函数。如果缺少 OVER 子句，则它是普通的聚合函数或标量函数。窗口函数还可能在函数和 OVER 子句之间有一个 FILTER 子句。

窗口函数的语法如下：

**窗口函数调用:**

<svg class="pikchr" viewBox="0 0 870.446 132.84"><text x="91" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口函数</text> <text x="182" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="276" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="370" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="482" y="85" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">过滤子句</text> <text x="610" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OVER</text> <text x="746" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口名称</text> <text x="742" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口定义</text> <text x="276" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="276" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text></svg>

**表达式:**

<svg class="pikchr" viewBox="0 0 963.96 1068.77"><text x="101" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">literal-value</text> <text x="116" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">bind-parameter</text> <text x="108" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">schema-name</text> <text x="210" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="313" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">table-name</text> <text x="404" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="518" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">column-name</text> <text x="114" y="153" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">unary-operator</text> <text x="231" y="153" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="189" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">binary-operator</text> <text x="308" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="111" y="230" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">function-name</text> <text x="209" y="230" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="348" y="230" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">function-arguments</text> <text x="488" y="230" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="597" y="260" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">filter-clause</text> <text x="783" y="260" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">over-clause</text> <text x="60" y="306" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="136" y="306" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="211" y="306" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="136" y="269" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="74" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CAST</text> <text x="141" y="343" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="204" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="269" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="358" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">type-name</text> <text x="446" y="343" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="69" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="164" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="302" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="69" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="155" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="261" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LIKE</text> <text x="264" y="457" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">GLOB</text> <text x="276" y="495" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">REGEXP</text> <text x="274" y="532" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">MATCH</text> <text x="403" y="495" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="403" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="506" y="449" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ESCAPE</text> <text x="595" y="449" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="570" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="170" y="570" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ISNULL</text> <text x="180" y="608" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOTNULL</text> <text x="155" y="646" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="234" y="646" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL</text> <text x="69" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="132" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IS</text> <text x="209" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="355" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="473" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FROM</text> <text x="566" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="155" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="268" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BETWEEN</text> <text x="367" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text

**function-arguments:**

<svg class="pikchr" viewBox="0 0 456.566 223.344"><text x="116" y="26" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="278" y="56" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="278" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="228" y="192" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="127" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="199" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="310" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ordering-term</text> <text x="310" y="154" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

**ordering-term:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DESC</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ASC</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FIRST</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LAST</text></svg>

**literal-value:**

<svg class="pikchr" viewBox="0 0 341.376 336.96"><text x="159" y="319" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前时间戳</text> <text x="119" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">数字文字</text> <text x="109" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">字符串文字</text> <text x="103" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">二进制大对象文字</text> <text x="81" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">空值</text> <text x="81" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">真</text> <text x="85" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">假</text> <text x="128" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前时间</text> <text x="129" y="281" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前日期</text></svg>

**over-clause:**

<svg class="pikchr" viewBox="0 0 600.706 418.392"><text x="62" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OVER</text> <text x="192" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">window-name</text> <text x="149" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="292" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">base-window-name</text> <text x="261" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分区</text> <text x="356" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="434" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="434" y="195" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="244" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序</text> <text x="321" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="439" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="439" y="309" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="258" y="384" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">帧规范</text> <text x="534" y="384" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**frame-spec:**

<svg class="pikchr" viewBox="0 0 1039.65 522.72"><text x="93" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">群组</text> <text x="258" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之间</text> <text x="417" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制的</text> <text x="559" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前置</text> <text x="685" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">和</text> <text x="817" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制的</text> <text x="961" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">后续</text> <text x="87" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">范围</text> <text x="83" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="272" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制的</text> <text x="420" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前置</text> <text x="231" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="338" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前置</text> <text x="257" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="357" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="482" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前置</text> <text x="401" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="501" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="484" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">后续</text> <text x="776" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="883" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前置</text> <text x="802" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="902" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="776" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="885" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">后续</text> <text x="495" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="617" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="717" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="495" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="604" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="495" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="593" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">绑定</text> <text x="495" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="586" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">其他</text> <text x="672" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">项</text></svg>

**排序条件:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">校对名称</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">降序</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">升序</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">空值</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">首位</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">空值</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">末位</text></svg>

**触发函数:**

<svg class="pikchr" viewBox="0 0 627.302 147.96"><text x="65" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">RAISE</text> <text x="135" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="246" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ROLLBACK</text> <text x="351" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="456" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">error-message</text> <text x="579" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="233" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">忽略</text> <text x="228" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">中止</text> <text x="218" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">失败</text></svg>

**选择语句:**

<svg class="pikchr" viewBox="0 0 669.677 1162.3"><text x="76" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WITH</text> <text x="210" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">RECURSIVE</text> <text x="471" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">公共表达式</text> <text x="471" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="167" y="129" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">SELECT</text> <text x="299" y="159" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="481" y="129" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">结果列</text> <text x="481" y="166" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="272" y="197" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ALL</text> <text x="186" y="272" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FROM</text> <text x="371" y="272" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="371" y="346" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接子句</text> <text x="371" y="309" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="193" y="422" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WHERE</text> <text x="280" y="422" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="190" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">GROUP</text> <text x="267" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="345" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="493" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">HAVING</text> <text x="582" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="345" y="558" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="200" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WINDOW</text> <text x="346" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口名称</text> <text x="450" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="550" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口定义</text> <text x="446" y="671" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="168" y="785" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">VALUES</text> <text x="260" y="785" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="336" y="785" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="412" y="785" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="336" y="747" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="336" y="823" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="336" y="876" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">复合操作符</text> <text x="667" y="275" text-anchor="middle" font-style="italic" fill="rgb(128,128,128)" transform="rotate(-90 667,285)" dominant-baseline="central">select-core</text> <text x="190" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="268" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="185" y="1057" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LIMIT</text> <text x="264" y="1057" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="395" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="395" y="989" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="389" y="1087" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OFFSET</text> <text x="477" y="1087" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="363" y="1125" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="426" y="1125" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text></svg>

**common-table-expression:**

<svg class="pikchr" viewBox="0 0 638.525 167.4"><text x="85" y="29" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="211" y="29" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="348" y="29" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列名</text> <text x="461" y="29" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="49" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="137" y="119" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">不</text> <text x="278" y="119" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">MATERIALIZED</text> <text x="410" y="150" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="500" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">select-stmt</text> <text x="591" y="150" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="348" y="66" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

**compound-operator:**

<svg class="pikchr" viewBox="0 0 293.842 147.96"><text x="105" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">UNION</text> <text x="105" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">UNION</text> <text x="125" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">INTERSECT</text> <text x="109" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">EXCEPT</text> <text x="187" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ALL</text></svg>

**join-clause:**

<svg class="pikchr" viewBox="0 0 793.282 84.24"><text x="112" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="319" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接操作符</text> <text x="483" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="654" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接约束条件</text></svg>

**join-constraint:**

<svg class="pikchr" viewBox="0 0 483.336 126.576"><text x="85" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">使用</text> <text x="158" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="271" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列名</text> <text x="384" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="271" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="70" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="137" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text></svg>

**连接操作符：**

<svg class="pikchr" viewBox="0 0 620.333 255.312"><text x="99" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">自然</text> <text x="259" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">左</text> <text x="415" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">外</text> <text x="543" y="41" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接</text> <text x="310" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="265" y="109" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">右</text> <text x="260" y="147" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">全</text> <text x="267" y="192" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">内</text> <text x="267" y="238" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">交叉</text></svg>

**排序项：**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">校对名称</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">降序</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">升序</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">首先</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">最后</text></svg>

**结果列：**

<svg class="pikchr" viewBox="0 0 398.054 163.08"><text x="69" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="153" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="265" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列别名</text> <text x="66" y="108" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="103" y="145" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="194" y="145" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="247" y="145" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text></svg>

**表格或子查询:**

<svg class="pikchr" viewBox="0 0 720.778 457.704"><text x="114" y="74" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">模式名称</text> <text x="209" y="74" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="319" y="36" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名称</text> <text x="425" y="36" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="527" y="36" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表别名</text> <text x="356" y="112" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">索引化</text> <text x="443" y="112" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="538" y="112" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">索引名称</text> <text x="333" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">未</text> <text x="429" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">索引化</text> <text x="356" y="213" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表函数名称</text> <text x="478" y="213" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="554" y="213" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="630" y="213" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="554" y="175" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="409" y="289" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="548" y="289" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表别名</text> <text x="66" y="327" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="156" y="327" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">选择语句</text> <text x="246" y="327" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="65" y="364" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="215" y="364" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="365" y="364" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="215" y="402" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="173" y="440" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接子句</text></svg>

**type-name:**

<svg class="pikchr" viewBox="0 0 661.008 110.16"><text x="74" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">名称</text> <text x="180" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="281" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数</text> <text x="382" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="484" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数</text> <text x="585" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="180" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="281" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数</text> <text x="382" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**有符号数:**

<svg class="pikchr" viewBox="0 0 292.013 99.576"><text x="66" y="44" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">+</text> <text x="191" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">数值文字</text> <text x="66" y="82" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">-</text></svg>

**过滤子句:**

<svg class="pikchr" viewBox="0 0 422.381 34.56"><text x="70" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">过滤</text> <text x="146" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="224" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">条件</text> <text x="312" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="374" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**窗口定义:**

<svg class="pikchr" viewBox="0 0 479.765 380.592"><text x="47" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="190" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">base-window-name</text> <text x="159" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分区</text> <text x="254" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="332" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="332" y="157" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="141" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序</text> <text x="219" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="337" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="337" y="271" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="156" y="346" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">框架规范</text> <text x="432" y="346" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**框架规范:**

<svg class="pikchr" viewBox="0 0 1039.65 522.72"><text x="93" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="258" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="417" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="559" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前导</text> <text x="685" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">和</text> <text x="817" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="961" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">后续</text> <text x="87" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">范围</text> <text x="83" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="272" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="420" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前导</text> <text x="231" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="338" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前导</text> <text x="257" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="357" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="482" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前导</text> <text x="401" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="501" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="484" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">后续</text> <text x="776" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="883" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">前导</text> <text x="802" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="902" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="776" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="885" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">后续</text> <text x="495" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="617" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="717" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="495" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="604" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分组</text> <text x="495" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="593" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">绑定</text> <text x="495" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="586" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">其他</text> <text x="672" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">内容</text></svg>

**ordering-term:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DESC</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ASC</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FIRST</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LAST</text></svg>

与普通函数不同，窗口函数不能使用 DISTINCT 关键字。此外，窗口函数只能出现在结果集和 SELECT 语句的 ORDER BY 子句中。

窗口函数分为两种类型：聚合窗口函数 和 内置窗口函数。每个聚合窗口函数也可以作为普通聚合函数工作，只需省略 OVER 和 FILTER 子句。此外，SQLite 的所有内置 聚合函数 都可以通过添加适当的 OVER 子句作为聚合窗口函数使用。应用程序可以使用 sqlite3_create_window_function() 接口注册新的聚合窗口函数。然而，内置窗口函数需要在查询规划器中进行特殊处理，因此应用程序无法添加具有内置窗口函数中特殊属性的新窗口函数。

这里是一个使用内置的 `row_number()` 窗口函数的示例：

```sql
CREATE TABLE t0(x INTEGER PRIMARY KEY, y TEXT);
INSERT INTO t0 VALUES (1, 'aaa'), (2, 'ccc'), (3, 'bbb');

*-- The following SELECT statement returns:*
*--* 
*--   x | y | row_number*
-----------------------
*--   1 | aaa | 1* 
*--   2 | ccc | 3* 
*--   3 | bbb | 2* 
*--* 
SELECT x, y, row_number() OVER (ORDER BY y) AS row_number FROM t0 ORDER BY x;

```

row_number() 窗口函数根据窗口定义中的 "ORDER BY" 子句（在本例中为 "ORDER BY y"）为每行分配连续整数。请注意，这不会影响从整体查询返回结果的顺序。最终输出的顺序仍由附加到 SELECT 语句的 ORDER BY 子句（在本例中为 "ORDER BY x"）控制。

命名的窗口定义子句也可以使用 WINDOW 子句添加到 SELECT 语句中，然后在窗口函数调用中按名称引用。例如，以下 SELECT 语句包含两个命名的窗口定义子句，"win1" 和 "win2"：

```sql
SELECT x, y, row_number() OVER win1, rank() OVER win2
FROM t0
WINDOW win1 AS (ORDER BY y RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW),
       win2 AS (PARTITION BY y ORDER BY x)
ORDER BY x;

```

WINDOW 子句（如果存在）位于任何 HAVING 子句之后和任何 ORDER BY 子句之前。

# 2\. 聚合窗口函数

本节中的示例都假设数据库填充如下：

```sql
CREATE TABLE t1(a INTEGER PRIMARY KEY, b, c);
INSERT INTO t1 VALUES   (1, 'A', 'one'  ),
                        (2, 'B', 'two'  ),
                        (3, 'C', 'three'),
                        (4, 'D', 'one'  ),
                        (5, 'E', 'two'  ),
                        (6, 'F', 'three'),
                        (7, 'G', 'one'  );

```

聚合窗口函数类似于普通聚合函数，但将其添加到查询不会更改返回的行数。 相反，对于每一行，聚合窗口函数的结果就像对 OVER 子句指定的“窗口框架”中的所有行运行相应的聚合函数一样。

```sql
*-- The following SELECT statement returns:*
*--* 
*--   a | b | group_concat*
-------------------------
*--   1 | A | A.B* 
*--   2 | B | A.B.C* 
*--   3 | C | B.C.D* 
*--   4 | D | C.D.E* 
*--   5 | E | D.E.F* 
*--   6 | F | E.F.G* 
*--   7 | G | F.G* 
*--* 
SELECT a, b, group_concat(b, '.') OVER (
  ORDER BY a ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
) AS group_concat FROM t1;

```

在上面的示例中，窗口框架由前一行（“1 PRECEDING”）和后一行（“1 FOLLOWING”）之间的所有行组成，包括根据窗口定义中的 ORDER BY 子句排序的行（在本例中为“ORDER BY a”）。 例如，具有(a=3)的行的框架包括行(2, 'B', 'two')，(3, 'C', 'three')和(4, 'D', 'one')。 因此，该行的 group_concat(b, '.')的结果是'B.C.D'。

所有 SQLite 的聚合函数都可以用作聚合窗口函数。 还可以创建用户定义的聚合窗口函数。

## 2.1\. PARTITION BY 子句

为了计算窗口函数，查询的结果集被划分为一个或多个“分区”。 分区由窗口定义中 PARTITION BY 子句的所有术语具有相同值的所有行组成。 如果没有 PARTITION BY 子句，则查询的整个结果集将是单个分区。 窗口函数的处理是针对每个分区单独进行的。

例如：

```sql
*-- The following SELECT statement returns:*
*--* 
*--   c     | a | b | group_concat*
---------------------------------
*--   one   | 1 | A | A.D.G* 
*--   one   | 4 | D | D.G* 
*--   one   | 7 | G | G* 
*--   three | 3 | C | C.F* 
*--   three | 6 | F | F* 
*--   two   | 2 | B | B.E* 
*--   two   | 5 | E | E* 
*--* 
SELECT c, a, b, group_concat(b, '.') OVER (
  PARTITION BY c ORDER BY a RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
) AS group_concat
FROM t1 ORDER BY c, a;

```

在上面的查询中，“PARTITION BY c”子句将结果集分为三个分区。 第一个分区有三行，c=='one'。 第二个分区有两行，c=='three'，第三个分区有两行，c=='two'。

在上面的示例中，每个分区的所有行最终都会在输出中分组在一起。 这是因为 PARTITION BY 子句是整个查询的 ORDER BY 子句的前缀。 但不一定是这种情况。 一个分区可以由结果集中随机散布的行组成。 例如：

```sql
*-- The following SELECT statement returns:*
*--* 
*--   c     | a | b | group_concat*
---------------------------------
*--   one   | 1 | A | A.D.G* 
*--   two   | 2 | B | B.E* 
*--   three | 3 | C | C.F* 
*--   one   | 4 | D | D.G* 
*--   two   | 5 | E | E* 
*--   three | 6 | F | F* 
*--   one   | 7 | G | G* 
*--* 
SELECT c, a, b, group_concat(b, '.') OVER (
  PARTITION BY c ORDER BY a RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
) AS group_concat
FROM t1 ORDER BY a;

```

## 2.2\. 框架规格

框架规格决定了聚合窗口函数读取哪些输出行。 框架规格由四个部分组成：

+   框架类型 - 可能是 ROWS、RANGE 或 GROUPS 中的一种，

+   一个起始框架边界，

+   一个结束框架边界，

+   一个 EXCLUDE 子句。

下面是语法细节：

**框架规格：**

<svg class="pikchr" viewBox="0 0 1039.65 522.72"><text x="93" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="258" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="417" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="559" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之间</text> <text x="685" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">和</text> <text x="817" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="961" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="87" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">范围</text> <text x="83" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="272" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="420" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="231" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="338" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="257" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="357" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="482" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="401" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="501" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="484" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="776" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="883" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="802" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="902" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="776" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="885" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="495" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="617" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="717" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="495" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="604" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="495" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="593" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">相等</text> <text x="495" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="586" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无</text> <text x="672" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">其他</text></svg>

**表达式:**

<svg class="pikchr" viewBox="0 0 963.96 1068.77"><text x="101" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">literal-value</text> <text x="116" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">bind-parameter</text> <text x="108" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">schema-name</text> <text x="210" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="313" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">table-name</text> <text x="404" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="518" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">column-name</text> <text x="114" y="153" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">unary-operator</text> <text x="231" y="153" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="189" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">binary-operator</text> <text x="308" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="111" y="230" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">function-name</text> <text x="209" y="230" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="348" y="230" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">function-arguments</text> <text x="488" y="230" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="597" y="260" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">filter-clause</text> <text x="783" y="260" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">over-clause</text> <text x="60" y="306" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="136" y="306" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="211" y="306" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="136" y="269" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="74" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CAST</text> <text x="141" y="343" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="204" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="269" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="358" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">type-name</text> <text x="446" y="343" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="69" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="164" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="302" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="69" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="155" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="261" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LIKE</text> <text x="264" y="457" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">GLOB</text> <text x="276" y="495" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">REGEXP</text> <text x="274" y="532" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">MATCH</text> <text x="403" y="495" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="403" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="506" y="449" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ESCAPE</text> <text x="595" y="449" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="570" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="170" y="570" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ISNULL</text> <text x="180" y="608" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOTNULL</text> <text x="155" y="646" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="234" y="646" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL</text> <text x="69" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="132" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IS</text> <text x="209" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="355" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="473" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FROM</text> <text x="566" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="155" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="268" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BETWEEN</text> <text x="367" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text

**filter-clause:**

<svg class="pikchr" viewBox="0 0 422.381 34.56"><text x="70" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FILTER</text> <text x="146" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="224" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WHERE</text> <text x="312" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="374" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**function-arguments:**

<svg class="pikchr" viewBox="0 0 456.566 223.344"><text x="116" y="26" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="278" y="56" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="278" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="228" y="192" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="127" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="199" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="310" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ordering-term</text> <text x="310" y="154" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

**ordering-term:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DESC</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ASC</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FIRST</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LAST</text></svg>

**literal-value:**

<svg class="pikchr" viewBox="0 0 341.376 336.96"><text x="159" y="319" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CURRENT_TIMESTAMP</text> <text x="119" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">numeric-literal</text> <text x="109" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">string-literal</text> <text x="103" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">blob-literal</text> <text x="81" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL</text> <text x="81" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">TRUE</text> <text x="85" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FALSE</text> <text x="128" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CURRENT_TIME</text> <text x="129" y="281" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CURRENT_DATE</text></svg>

**over-clause:**

<svg class="pikchr" viewBox="0 0 600.706 418.392"><text x="62" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OVER</text> <text x="192" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">window-name</text> <text x="149" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="292" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">base-window-name</text> <text x="261" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">PARTITION</text> <text x="356" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="434" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="434" y="195" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="244" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="321" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="439" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ordering-term</text> <text x="439" y="309" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="258" y="384" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">frame-spec</text> <text x="534" y="384" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**ordering-term:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DESC</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ASC</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FIRST</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LAST</text></svg>

**raise-function:**

<svg class="pikchr" viewBox="0 0 627.302 147.96"><text x="65" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">RAISE</text> <text x="135" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="246" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ROLLBACK</text> <text x="351" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="456" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">error-message</text> <text x="579" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="233" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IGNORE</text> <text x="228" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ABORT</text> <text x="218" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FAIL</text></svg>

**select-stmt:**

<svg class="pikchr" viewBox="0 0 669.677 1162.3"><text x="76" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WITH</text> <text x="210" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">RECURSIVE</text> <text x="471" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">common-table-expression</text> <text x="471" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="167" y="129" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">SELECT</text> <text x="299" y="159" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="481" y="129" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">result-column</text> <text x="481" y="166" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="272" y="197" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ALL</text> <text x="186" y="272" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FROM</text> <text x="371" y="272" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">table-or-subquery</text> <text x="371" y="346" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">join-clause</text> <text x="371" y="309" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="193" y="422" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WHERE</text> <text x="280" y="422" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="190" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">GROUP</text> <text x="267" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="345" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="493" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">HAVING</text> <text x="582" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="345" y="558" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="200" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WINDOW</text> <text x="346" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">window-name</text> <text x="450" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="550" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">window-defn</text> <text x="446" y="671" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="168" y="785" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">VALUES</text> <text x="260" y="785" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="336" y="785" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="412" y="785" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="336" y="747" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="336" y="823" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="336" y="876" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">compound-operator</text> <text x="667" y="275" text-anchor="middle" font-style="italic" fill="rgb(128,128,128)" transform="rotate(-90 667,285)" dominant-baseline="central">select-core</text> <text x="190" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="268" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="185" y="1057" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LIMIT</text> <text x="264" y="1057" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="395" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ordering-term</text> <text x="395" y="989" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="389" y="1087" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OFFSET</text> <text x="477" y="1087" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="363" y="1125" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="426" y="1125" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text></svg>

**公共表达式:**

<svg class="pikchr" viewBox="0 0 638.525 167.4"><text x="85" y="29" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="211" y="29" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="348" y="29" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列名</text> <text x="461" y="29" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="49" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">作为</text> <text x="137" y="119" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">非</text> <text x="278" y="119" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">物化</text> <text x="410" y="150" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="500" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">选择语句</text> <text x="591" y="150" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="348" y="66" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

**复合操作符:**

<svg class="pikchr" viewBox="0 0 293.842 147.96"><text x="105" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">联合</text> <text x="105" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">联合</text> <text x="125" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">交集</text> <text x="109" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">差集</text> <text x="187" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">全部</text></svg>

**连接子句:**

<svg class="pikchr" viewBox="0 0 793.282 84.24"><text x="112" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="319" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接操作符</text> <text x="483" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="654" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接约束</text></svg>

**连接约束:**

<svg class="pikchr" viewBox="0 0 483.336 126.576"><text x="85" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">使用</text> <text x="158" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="271" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列名</text> <text x="384" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="271" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="70" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="137" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text></svg>

**连接运算符:**

<svg class="pikchr" viewBox="0 0 620.333 255.312"><text x="99" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">自然</text> <text x="259" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">左</text> <text x="415" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">外连接</text> <text x="543" y="41" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">联接</text> <text x="310" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="265" y="109" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">右</text> <text x="260" y="147" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">全</text> <text x="267" y="192" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">内</text> <text x="267" y="238" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">交叉</text></svg>

**排序项:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序规则</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序规则名称</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">降序</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">升序</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">空值</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">最前</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">空值</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">最后</text></svg>

**结果列:**

<svg class="pikchr" viewBox="0 0 398.054 163.08"><text x="69" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="153" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="265" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列别名</text> <text x="66" y="108" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="103" y="145" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="194" y="145" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="247" y="145" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text></svg>

**表或子查询:**

**window-defn:**

**窗口定义:**

<svg class="pikchr" viewBox="0 0 479.765 380.592"><text x="47" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="190" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">base-window-name</text> <text x="159" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分区</text> <text x="254" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="332" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="332" y="157" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="141" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序</text> <text x="219" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="337" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="337" y="271" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="156" y="346" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">帧规范</text> <text x="432" y="346" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**类型名称:**

<svg class="pikchr" viewBox="0 0 661.008 110.16"><text x="74" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">名称</text> <text x="180" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="281" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数字</text> <text x="382" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="484" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数字</text> <text x="585" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="180" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="281" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数字</text> <text x="382" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**有符号数字:**

<svg class="pikchr" viewBox="0 0 292.013 99.576"><text x="66" y="44" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">+</text> <text x="191" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">数值文字</text> <text x="66" y="82" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">-</text></svg>

结束框架边界可以省略（如果省略了围绕起始框架边界的 BETWEEN 和 AND 关键字），此时结束框架边界默认为 CURRENT ROW。

如果框架类型为 RANGE 或 GROUPS，则具有相同 ORDER BY 表达式值的行被视为 "同级"。或者，如果没有 ORDER BY 项，则所有行都是同级的。同级始终位于同一个框架内。

默认框架规范是：

```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE NO OTHERS

```

默认意味着聚合窗口函数读取分区中从开头到当前行及其同级的所有行。这意味着对于所有 ORDER BY 表达式具有相同值的行，窗口函数的结果值也将相同（因为窗口框架相同）。例如：

```sql
*-- The following SELECT statement returns:*
*--* 
*--   a | b | c | group_concat*
-----------------------------
*--   1 | A | one   | A.D.G* 
*--   2 | B | two   | A.D.G.C.F.B.E*
*--   3 | C | three | A.D.G.C.F* 
*--   4 | D | one   | A.D.G* 
*--   5 | E | two   | A.D.G.C.F.B.E*
*--   6 | F | three | A.D.G.C.F* 
*--   7 | G | one   | A.D.G* 
*--* 
SELECT a, b, c,
       group_concat(b, '.') OVER (ORDER BY c) AS group_concat
FROM t1 ORDER BY a;

```

### 2.2.1\. 框架类型

有三种框架类型：ROWS、GROUPS 和 RANGE。框架类型确定了框架的起始和结束边界如何度量。

+   **ROWS**：ROWS 框架类型意味着框架的起始和结束边界是相对于当前行计算的个别行数。

+   **GROUPS**：GROUPS 框架类型意味着起始和结束边界是相对于当前组计算的 "组" 数。一个 "组" 是所有窗口 ORDER BY 子句的所有项都具有等效值的行的集合。换句话说，一个组包含所有与某行同级的行。

+   **RANGE**：RANGE 框架类型要求窗口的 ORDER BY 子句恰好有一个项。将该项称为 "X"。使用 RANGE 框架类型时，通过计算分区中所有行的表达式 X 的值，并为那些其 X 的值在当前行的 X 值的某个范围内的行进行分组来确定框架的元素。有关详细信息，请参阅下面 "<expr> PRECEDING" 边界规范的描述。

ROWS 和 GROUPS 框架类型相似，因为它们都通过相对于当前行计算来确定框架的范围。不同之处在于 ROWS 计算个别行数，而 GROUPS 计算同级组。RANGE 框架类型不同，它通过查找相对于当前行某个值带有一定值段的表达式值来确定框架的范围。

### 2.2.2\. 框架边界

有五种描述起始和结束框架边界的方式：

1.  **UNBOUNDED PRECEDING**

    框架边界是分区中的第一行（partition）。

1.  **<expr> PRECEDING**

    <expr> 必须是一个非负常数数值表达式。边界是当前行之前 <expr> "单位" 的行。这里 "单位" 的含义取决于框架类型：

    +   **行 →** 框架边界是距离当前行 <expr> 行的行，或者如果在当前行之前的行数少于 <expr> 行，则是分区的第一行。 <expr> 必须是整数。

    +   **组 →** "组" 是一组对等行 - 所有这些行在 ORDER BY 子句的每个项中都具有相同的值。框架边界是距离包含当前行的组的第 <expr> 组之前的组，或者如果在当前行之前的组数少于 <expr> 组，则是分区的第一组。对于框架的起始边界，使用组的第一行，并且对于框架的结束边界，使用组的最后一行。 <expr> 必须是整数。

    +   **范围 →** 对于此形式，window-defn 的 ORDER BY 子句必须有一个单独的项。称该 ORDER BY 项为 "X"。设 X[i] 为分区中第 i 行的 X 表达式的值，X[c] 为当前行的 X 的值。非正式地说，范围边界是第一行，其中 X[i] 在 X[c] 的 <expr> 内。更确切地说：

        1.  如果 X[i] 或 X[c] 是非数值的，则边界是使表达式 "X[i] IS X[c]" 为真的第一行。

        1.  如果 ORDER BY 是 ASC，则边界是第一行，其 X[i]>=X[c]-<expr>。

        1.  如果 ORDER BY 是 DESC，则边界是第一行，其 X[i]<=X[c]+<expr>。对于此形式，<expr> 不必是整数。只要它是常数且非负，它可以求值为实数。"0 前导" 的边界描述始终意味着与 "当前行" 相同的含义。

1.  **当前行**

    当前行。对于 RANGE 和 GROUPS 框架类型，当前行的对等行也包括在框架中，除非明确由 EXCLUDE 子句排除。无论是否将 CURRENT ROW 用作起始或结束框架边界，这都是正确的。

1.  **<expr> 后续**

    这与 "<expr> 前导" 相同，只是边界是当前行之后 <expr> 个单位，而不是之前。

1.  **未限定后续**

    框架边界是 partition 中的最后一行。

结束框架边界可能不采用出现在上述列表中较高的形式作为起始框架边界。

在下面的示例中，每行的窗口框架包括从当前行到集合末尾的所有行，其中按 "ORDER BY a" 排序。

```sql
*-- The following SELECT statement returns:*
*--* 
*--   c     | a | b | group_concat*
---------------------------------
*--   one   | 1 | A | A.D.G.C.F.B.E*
*--   one   | 4 | D | D.G.C.F.B.E* 
*--   one   | 7 | G | G.C.F.B.E* 
*--   three | 3 | C | C.F.B.E* 
*--   three | 6 | F | F.B.E* 
*--   two   | 2 | B | B.E* 
*--   two   | 5 | E | E* 
*--* 
SELECT c, a, b, group_concat(b, '.') OVER (
  ORDER BY c, a ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
) AS group_concat
FROM t1 ORDER BY c, a;

```

### 2.2.3\. EXCLUDE 子句

可选的 EXCLUDE 子句可以采用以下四种形式之一：

+   **排除无其他**：这是默认值。在这种情况下，窗口框架不排除任何行，如其起始和结束框架边界所定义的那样。

+   **排除当前行**：在这种情况下，当前行被排除在窗口框架之外。对于 GROUPS 和 RANGE 框架类型，当前行的对等行仍保留在框架中。

+   **排除 GROUP**：在这种情况下，当前行及其所有同级行都被排除在框架之外。处理 EXCLUDE 子句时，所有具有相同 ORDER BY 值的行，或者如果没有 ORDER BY 子句，则分区中的所有行都被视为同级，即使框架类型是 ROWS。

+   **排除 TIES**：在这种情况下，当前行是框架的一部分，但当前行的同级被排除。

下面的示例演示了 EXCLUDE 子句的各种形式的效果：

```sql
*-- The following SELECT statement returns:*
*--* 
*--   c    | a | b | no_others     | current_row | grp       | ties*
*--  one   | 1 | A | A.D.G         | D.G         |           | A*
*--  one   | 4 | D | A.D.G         | A.G         |           | D*
*--  one   | 7 | G | A.D.G         | A.D         |           | G*
*--  three | 3 | C | A.D.G.C.F     | A.D.G.F     | A.D.G     | A.D.G.C*
*--  three | 6 | F | A.D.G.C.F     | A.D.G.C     | A.D.G     | A.D.G.F*
*--  two   | 2 | B | A.D.G.C.F.B.E | A.D.G.C.F.E | A.D.G.C.F | A.D.G.C.F.B*
*--  two   | 5 | E | A.D.G.C.F.B.E | A.D.G.C.F.B | A.D.G.C.F | A.D.G.C.F.E*
*--* 
SELECT c, a, b,
  group_concat(b, '.') OVER (
    ORDER BY c GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE NO OTHERS
  ) AS no_others,
  group_concat(b, '.') OVER (
    ORDER BY c GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE CURRENT ROW
  ) AS current_row,
  group_concat(b, '.') OVER (
    ORDER BY c GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE GROUP
  ) AS grp,
  group_concat(b, '.') OVER (
    ORDER BY c GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW EXCLUDE TIES
  ) AS ties
FROM t1 ORDER BY c, a;

```

## 2.3\. 过滤子句

**filter-clause:**

<svg class="pikchr" viewBox="0 0 422.381 34.56"><text x="70" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">过滤</text> <text x="146" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="224" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="312" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="374" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">中</text></svg>

**expr:**

<svg class="pikchr" viewBox="0 0 963.96 1068.77"><text x="101" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">literal-value</text> <text x="116" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">bind-parameter</text> <text x="108" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">schema-name</text> <text x="210" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="313" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">table-name</text> <text x="404" y="115" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="518" y="115" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">column-name</text> <text x="114" y="153" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">unary-operator</text> <text x="231" y="153" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="189" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">binary-operator</text> <text x="308" y="191" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="111" y="230" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">function-name</text> <text x="209" y="230" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="348" y="230" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">function-arguments</text> <text x="488" y="230" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="597" y="260" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">filter-clause</text> <text x="783" y="260" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">over-clause</text> <text x="60" y="306" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="136" y="306" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="211" y="306" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="136" y="269" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="74" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CAST</text> <text x="141" y="343" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="204" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="269" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="358" y="343" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">type-name</text> <text x="446" y="343" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="69" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="164" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="302" y="381" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="69" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="155" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="261" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LIKE</text> <text x="264" y="457" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">GLOB</text> <text x="276" y="495" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">REGEXP</text> <text x="274" y="532" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">MATCH</text> <text x="403" y="495" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="403" y="419" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="506" y="449" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ESCAPE</text> <text x="595" y="449" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="570" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="170" y="570" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ISNULL</text> <text x="180" y="608" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOTNULL</text> <text x="155" y="646" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="234" y="646" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL</text> <text x="69" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="132" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IS</text> <text x="209" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="355" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="473" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FROM</text> <text x="566" y="684" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="69" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="155" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="268" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BETWEEN</text> <text x="367" y="729" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text

**function-arguments:**

<svg class="pikchr" viewBox="0 0 456.566 223.344"><text x="116" y="26" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="278" y="56" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="278" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="228" y="192" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="127" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="199" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="310" y="117" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ordering-term</text> <text x="310" y="154" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

**ordering-term:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">expr</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">collation-name</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DESC</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ASC</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FIRST</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LAST</text></svg>

**literal-value:**

<svg class="pikchr" viewBox="0 0 341.376 336.96"><text x="159" y="319" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CURRENT_TIMESTAMP</text> <text x="119" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">numeric-literal</text> <text x="109" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">string-literal</text> <text x="103" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">blob-literal</text> <text x="81" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL</text> <text x="81" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">TRUE</text> <text x="85" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FALSE</text> <text x="128" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CURRENT_TIME</text> <text x="129" y="281" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CURRENT_DATE</text></svg>

**over-clause:**

<svg class="pikchr" viewBox="0 0 600.706 418.392"><text x="62" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OVER</text> <text x="192" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口名称</text> <text x="149" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="292" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">基础窗口名称</text> <text x="261" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="356" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="434" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分区</text> <text x="434" y="195" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="244" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="321" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="439" y="271" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序</text> <text x="439" y="309" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="258" y="384" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">帧规范</text> <text x="534" y="384" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**frame-spec:**

<svg class="pikchr" viewBox="0 0 1039.65 522.72"><text x="93" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="258" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之间</text> <text x="417" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制</text> <text x="559" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="685" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">和</text> <text x="817" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后的无限制</text> <text x="961" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">范围行</text> <text x="87" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">范围</text> <text x="83" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="272" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限制之前</text> <text x="420" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="231" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="338" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="257" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前行</text> <text x="357" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="482" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="401" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前行</text> <text x="501" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="484" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="776" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="883" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="802" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前行</text> <text x="902" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="776" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="885" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="495" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="617" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="717" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="495" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="604" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="495" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="593" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">并列</text> <text x="495" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="586" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">其他</text> <text x="672" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">人</text></svg>

**排序项:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序规则名称</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">降序</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">升序</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL 值优先</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL 值最后</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL 值优先</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULL 值最后</text></svg>

**提升函数:**

<svg class="pikchr" viewBox="0 0 627.302 147.96"><text x="65" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">提升</text> <text x="135" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">（</text> <text x="246" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">回滚</text> <text x="351" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">，</text> <text x="456" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">错误消息</text> <text x="579" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">）</text> <text x="233" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">忽略</text> <text x="228" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">中止</text> <text x="218" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">失败</text></svg>

**选择语句:**

<svg class="pikchr" viewBox="0 0 669.677 1162.3"><text x="76" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WITH</text> <text x="210" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">RECURSIVE</text> <text x="471" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">公共表达式</text> <text x="471" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="167" y="129" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">SELECT</text> <text x="299" y="159" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">DISTINCT</text> <text x="481" y="129" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">结果列</text> <text x="481" y="166" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="272" y="197" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ALL</text> <text x="186" y="272" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">FROM</text> <text x="371" y="272" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="371" y="346" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接子句</text> <text x="371" y="309" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="193" y="422" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WHERE</text> <text x="280" y="422" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="190" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">GROUP</text> <text x="267" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="345" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="493" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">HAVING</text> <text x="582" y="520" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="345" y="558" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="200" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">WINDOW</text> <text x="346" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口名称</text> <text x="450" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="550" y="634" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">窗口定义</text> <text x="446" y="671" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="168" y="785" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">VALUES</text> <text x="260" y="785" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="336" y="785" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="412" y="785" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="336" y="747" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="336" y="823" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="336" y="876" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">复合操作符</text> <text x="667" y="275" text-anchor="middle" font-style="italic" fill="rgb(128,128,128)" transform="rotate(-90 667,285)" dominant-baseline="central">核心查询</text> <text x="190" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">ORDER</text> <text x="268" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">BY</text> <text x="185" y="1057" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">LIMIT</text> <text x="264" y="1057" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="395" y="951" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="395" y="989" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="389" y="1087" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">OFFSET</text> <text x="477" y="1087" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="363" y="1125" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="426" y="1125" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text></svg>

**common-table-expression:**

<svg class="pikchr" viewBox="0 0 638.525 167.4"><text x="85" y="29" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="211" y="29" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="348" y="29" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列名</text> <text x="461" y="29" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="49" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">作为</text> <text x="137" y="119" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">不</text> <text x="278" y="119" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">材料化</text> <text x="410" y="150" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="500" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">选择语句</text> <text x="591" y="150" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="348" y="66" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

**compound-operator:**

<svg class="pikchr" viewBox="0 0 293.842 147.96"><text x="105" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">联合</text> <text x="105" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">联合</text> <text x="125" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">交集</text> <text x="109" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">差集</text> <text x="187" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">全部</text></svg>

**join-clause:**

<svg class="pikchr" viewBox="0 0 793.282 84.24"><text x="112" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="319" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接操作符</text> <text x="483" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="654" y="47" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接约束</text></svg>

**join-constraint:**

<svg class="pikchr" viewBox="0 0 483.336 126.576"><text x="85" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">使用</text> <text x="158" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="271" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列名</text> <text x="384" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="271" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="70" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="137" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text></svg>

**join-operator:**

<svg class="pikchr" viewBox="0 0 620.333 255.312"><text x="99" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">自然</text> <text x="259" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">左</text> <text x="415" y="71" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">外部</text> <text x="543" y="41" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接</text> <text x="310" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="265" y="109" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">右</text> <text x="260" y="147" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">完全</text> <text x="267" y="192" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">内部</text> <text x="267" y="238" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">交叉</text></svg>

**ordering-term:**

<svg class="pikchr" viewBox="0 0 798.451 99.576"><text x="56" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="158" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">COLLATE</text> <text x="296" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序规则名称</text> <text x="477" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">降序</text> <text x="471" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">升序</text> <text x="627" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="718" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">优先</text> <text x="627" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NULLS</text> <text x="714" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">最后</text></svg>

**result-column:**

<svg class="pikchr" viewBox="0 0 398.054 163.08"><text x="69" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="153" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="265" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">列别名</text> <text x="66" y="108" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text> <text x="103" y="145" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="194" y="145" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="247" y="145" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text></svg>

**表或子查询:**

<svg class="pikchr" viewBox="0 0 720.778 457.704"><text x="114" y="74" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">模式名</text> <text x="209" y="74" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="319" y="36" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="425" y="36" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="527" y="36" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表别名</text> <text x="356" y="112" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="443" y="112" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">索引</text> <text x="538" y="112" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">索引名</text> <text x="333" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">未</text> <text x="429" y="150" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">索引</text> <text x="356" y="213" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表函数名</text> <text x="478" y="213" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">（</text> <text x="554" y="213" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="630" y="213" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">）</text> <text x="554" y="175" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">，</text> <text x="409" y="289" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">AS</text> <text x="548" y="289" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表别名</text> <text x="66" y="327" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">（</text> <text x="156" y="327" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">选择语句</text> <text x="246" y="327" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">）</text> <text x="65" y="364" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">（</text> <text x="215" y="364" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表或子查询</text> <text x="365" y="364" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">）</text> <text x="215" y="402" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">，</text> <text x="173" y="440" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接子句</text></svg>

**窗口定义:**

<svg class="pikchr" viewBox="0 0 479.765 380.592"><text x="47" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="190" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">base-window-name</text> <text x="159" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">分区</text> <text x="254" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="332" y="120" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="332" y="157" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="141" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">按</text> <text x="219" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序</text> <text x="337" y="233" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排序项</text> <text x="337" y="271" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="156" y="346" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">框架规范</text> <text x="432" y="346" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**框架规范:**

<svg class="pikchr" viewBox="0 0 1039.65 522.72"><text x="93" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="258" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">在</text> <text x="417" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限</text> <text x="559" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="685" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">和</text> <text x="817" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限</text> <text x="961" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="87" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">范围</text> <text x="83" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="272" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无限</text> <text x="420" y="168" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="231" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="338" y="206" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="257" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="357" y="244" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="482" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="401" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="501" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="375" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="484" y="130" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="776" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="883" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之前</text> <text x="802" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="902" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="776" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="885" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">之后</text> <text x="495" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="617" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">当前</text> <text x="717" y="395" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">行</text> <text x="495" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="604" y="433" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">组</text> <text x="495" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="593" y="470" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">绑定</text> <text x="495" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">排除</text> <text x="586" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">无</text> <text x="672" y="357" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">其他</text></svg>

**type-name:**

<svg class="pikchr" viewBox="0 0 661.008 110.16"><text x="74" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">名称</text> <text x="180" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="281" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数</text> <text x="382" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="484" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数</text> <text x="585" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="180" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="281" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">有符号数</text> <text x="382" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text></svg>

**signed-number:**

<svg class="pikchr" viewBox="0 0 292.013 99.576"><text x="66" y="44" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">+</text> <text x="191" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">数值字面量</text> <text x="66" y="82" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">-</text></svg>

如果提供了 FILTER 子句，则仅包括 *expr* 为 true 的行在窗口框架中。聚合窗口仍然为每一行返回一个值，但对于 FILTER 表达式求值为 true 以外的行，则不包括在任何行的窗口框架中。例如：

```sql
*-- The following SELECT statement returns:*
*--* 
*--   c     | a | b | group_concat*
---------------------------------
*--   one   | 1 | A | A* 
*--   two   | 2 | B | A* 
*--   three | 3 | C | A.C* 
*--   one   | 4 | D | A.C.D* 
*--   two   | 5 | E | A.C.D* 
*--   three | 6 | F | A.C.D.F* 
*--   one   | 7 | G | A.C.D.F.G* 
*--* 
SELECT c, a, b, group_concat(b, '.') FILTER (WHERE c!='two') OVER (
  ORDER BY a
) AS group_concat
FROM t1 ORDER BY a;

```

# 3\. 内置窗口函数

SQLite 不仅支持聚合窗口函数，还提供了一组基于[PostgreSQL 支持的内置窗口函数](https://www.postgresql.org/docs/10/static/functions-window.html)的函数。

内置窗口函数与聚合窗口函数一样，遵循任何 PARTITION BY 子句 - 每个选定的行被分配到一个分区并分别处理。每个内置窗口函数受 ORDER BY 子句影响的方式在下面描述。某些窗口函数（rank()、dense_rank()、percent_rank() 和 ntile()）使用“对等组”的概念（在同一分区内具有所有 ORDER BY 表达式相同值的行）。在这些情况下，不管帧规范指定 ROWS、GROUPS 还是 RANGE，对于内置窗口函数处理来说，具有相同所有 ORDER BY 表达式值的行被视为对等组成员。

大多数内置窗口函数忽略帧规范，例外包括 first_value()、last_value()和 nth_value()。在内置窗口函数调用的一部分指定 FILTER 子句是语法错误。

SQLite 支持以下 11 个内置窗口函数：

**row_number()**

当前分区内的行数。行号从窗口定义中的 ORDER BY 子句开始，按顺序编号，否则按任意顺序。

**rank()**

每个组中第一个同级行的 row_number() - 当前行的排名，有间隙。如果没有 ORDER BY 子句，则所有行都被视为同级，此函数始终返回 1。

**dense_rank()**

当前行在其分区内的同级群体编号 - 当前行的排名，没有间隙。行号从窗口定义中的 ORDER BY 子句开始。如果没有 ORDER BY 子句，则所有行都被视为同级，此函数始终返回 1。

**percent_rank()**

尽管名称如此，此函数始终返回一个值，介于 0.0 和 1.0 之间，等于（*rank* - 1）/（*partition-rows* - 1），其中*rank*是内置窗口函数 rank()返回的值，*partition-rows*是分区中的总行数。如果分区仅包含一行，则此函数返回 0.0。

**cume_dist()**

累积分布。计算为*row-number*/*partition-rows*，其中*row-number*是对最后一个同级组的 row_number()返回的值，*partition-rows*是分区中的行数。

**ntile(N)**

参数*N*被处理为整数。此函数尽可能均匀地将分区分成 N 组，并为每个组分配一个介于 1 和*N*之间的整数，按 ORDER BY 子句定义的顺序或任意顺序。必要时，较大的组首先出现。此函数返回分配给当前行所属组的整数值。

**lag(expr)

lag(expr, offset)

lag(expr, offset, default)**

lag()函数的第一种形式返回对分区中前一行评估表达式*expr*的结果。或者，如果没有前一行（因为当前行是第一行），则返回 NULL。

如果提供了*offset*参数，则必须是非负整数。在这种情况下，返回的值是对当前行之前分区内*offset*行评估*expr*的结果。如果*offset*为 0，则对当前行评估*expr*。如果当前行之前没有*offset*行，则返回 NULL。

如果还提供了*default*，则在由*offset*标识的行不存在时，返回它而不是 NULL。

**lead(expr)

lead(expr, offset)

lead(expr, offset, default)**

lead()函数的第一种形式返回对分区中下一行评估表达式*expr*的结果。或者，如果没有下一行（因为当前行是最后一行），则返回 NULL。

如果提供了*offset*参数，则它必须是非负整数。在这种情况下，返回的值是对分区内当前行后*offset*行评估*expr*的结果。如果*offset*为 0，则对当前行评估*expr*。如果当前行后没有*offset*行，则返回 NULL。

如果提供了*default*，则在*offset*指定的行不存在时，返回*default*而不是 NULL。

**first_value(expr)**

此内置窗口函数计算每行的窗口框架，方式与聚合窗口函数相同。它返回对窗口框架中第一行评估的*expr*的值。

**last_value(expr)**

此内置窗口函数计算每行的窗口框架，方式与聚合窗口函数相同。它返回对窗口框架中最后一行评估的*expr*的值。

**nth_value(expr, N)**

此内置窗口函数计算每行的窗口框架，方式与聚合窗口函数相同。它返回对窗口框架中第*N*行评估的*expr*的值。行在窗口框架内按 ORDER BY 子句定义的顺序从 1 开始编号，如果有的话，否则按任意顺序。如果分区中没有第*N*行，则返回 NULL。

本节的示例使用先前定义的 T1 表以及以下 T2 表：

```sql
CREATE TABLE t2(a, b);
INSERT INTO t2 VALUES('a', 'one'),
                     ('a', 'two'),
                     ('a', 'three'),
                     ('b', 'four'),
                     ('c', 'five'),
                     ('c', 'six');

```

下面的示例说明了五个排名函数的行为 - row_number()、rank()、dense_rank()、percent_rank()和 cume_dist()。

```sql
*-- The following SELECT statement returns:*
*--* 
*--   a | row_number | rank | dense_rank | percent_rank | cume_dist*
------------------------------------------------------------------
*--   a |          1 |    1 |          1 |          0.0 |       0.5*
*--   a |          2 |    1 |          1 |          0.0 |       0.5*
*--   a |          3 |    1 |          1 |          0.0 |       0.5*
*--   b |          4 |    4 |          2 |          0.6 |       0.66*
*--   c |          5 |    5 |          3 |          0.8 |       1.0*
*--   c |          6 |    5 |          3 |          0.8 |       1.0*
*--* 
SELECT a                        AS a,
       row_number() OVER win    AS row_number,
       rank() OVER win          AS rank,
       dense_rank() OVER win    AS dense_rank,
       percent_rank() OVER win  AS percent_rank,
       cume_dist() OVER win     AS cume_dist
FROM t2
WINDOW win AS (ORDER BY a);

```

下面的示例使用 ntile()将六行划分为两组（ntile(2)调用）和四组（ntile(4)调用）。对于 ntile(2)，每组分配了三行。对于 ntile(4)，有两组两行和两组一行。较大的两组先出现。

```sql
*-- The following SELECT statement returns:*
*--* 
*--   a | b     | ntile_2 | ntile_4*
----------------------------------
*--   a | one   |       1 |       1*
*--   a | two   |       1 |       1*
*--   a | three |       1 |       2*
*--   b | four  |       2 |       2*
*--   c | five  |       2 |       3*
*--   c | six   |       2 |       4*
*--* 
SELECT a                        AS a,
       b                        AS b,
       ntile(2) OVER win        AS ntile_2,
       ntile(4) OVER win        AS ntile_4
FROM t2
WINDOW win AS (ORDER BY a);

```

下一个示例演示了 lag()、lead()、first_value()、last_value()和 nth_value()。lag()和 lead()都忽略 frame-spec，但 first_value()、last_value()和 nth_value()遵循它。

```sql
*-- The following SELECT statement returns:*
*--* 
*--   b | lead | lag  | first_value | last_value | nth_value_3*
-------------------------------------------------------------
*--   A | C    | NULL | A           | A          | NULL* 
*--   B | D    | A    | A           | B          | NULL* 
*--   C | E    | B    | A           | C          | C* 
*--   D | F    | C    | A           | D          | C* 
*--   E | G    | D    | A           | E          | C* 
*--   F | n/a  | E    | A           | F          | C* 
*--   G | n/a  | F    | A           | G          | C* 
*--* 
SELECT b                          AS b,
       lead(b, 2, 'n/a') OVER win AS lead,
       lag(b) OVER win            AS lag,
       first_value(b) OVER win    AS first_value,
       last_value(b) OVER win     AS last_value,
       nth_value(b, 3) OVER win   AS nth_value_3
FROM t1
WINDOW win AS (ORDER BY b ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

```

# 4\. 窗口链

窗口链是一种简写，允许一个窗口以另一个窗口为基础定义。具体而言，此简写允许新窗口隐式复制基础窗口的 PARTITION BY 子句，并可选地复制 ORDER BY 子句。例如，在以下示例中：

```sql
SELECT group_concat(b, '.') OVER (
  win ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
FROM t1
WINDOW win AS (PARTITION BY a ORDER BY c)

```

group_concat()函数使用的窗口等效于"PARTITION BY a ORDER BY c ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW"。为了使用窗口链接，必须满足以下所有条件：

+   新的窗口定义不应包含 PARTITION BY 子句。如果有 PARTITION BY 子句，必须由基本窗口规范提供。

+   如果基本窗口有 ORDER BY 子句，则将其复制到新窗口中。在这种情况下，新窗口不得指定 ORDER BY 子句。如果基本窗口没有 ORDER BY 子句，则可以作为新窗口定义的一部分指定。

+   基本窗口可能不指定帧规范。帧规范只能在新窗口规范中给出。

下面的两个 SQL 片段相似，但不完全等效，因为如果窗口 "win" 的定义包含帧规范，则后者将失败。

```sql
SELECT group_concat(b, '.') OVER win ...
SELECT group_concat(b, '.') OVER (win) ...

```

# 5\. 用户定义的聚合窗口函数

可以使用 sqlite3_create_window_function() API 创建用户定义的聚合窗口函数。实现聚合窗口函数与普通聚合函数非常类似。任何用户定义的聚合窗口函数也可以用作普通聚合。为了实现用户定义的聚合窗口函数，应用程序必须提供四个回调函数：

| 回调 | 描述 |
| --- | --- |
| xStep | 此方法被窗口聚合和传统聚合函数实现所需。调用它以将行添加到当前窗口。如果有的话，传递给 xStep 实现的参数与要添加的行相对应。 |
| xFinal | 此方法被窗口聚合和传统聚合函数实现所需。调用它以返回聚合的当前值（由当前窗口的内容确定），并释放早期对 xStep 的调用分配的任何资源。 |
| xValue | 此方法仅适用于窗口聚合函数。此方法的存在区分窗口聚合函数与传统聚合函数。调用此方法以返回聚合的当前值。与 xFinal 不同，实现不应删除任何上下文。 |
| xInverse | 此方法仅适用于窗口聚合函数，而不适用于传统聚合函数实现。它被调用以从当前窗口中删除 xStep 目前聚合结果中的最旧结果。如果有，则函数参数是传递给删除行的 xStep 的参数。 |

下面的 C 代码实现了一个名为 sumint() 的简单窗口聚合函数。它的工作方式与内置的 sum() 函数相同，但是如果传递的参数不是整数值，则会抛出异常。

```sql

```

*/**

*** sumint() 的 xStep。*

****

*** 将参数的值添加到聚合上下文中（一个整数）。*

**/*

static void sumintStep(

sqlite3_context *ctx,

int nArg,

sqlite3_value *apArg[]

){

sqlite3_int64 *pInt;

assert( nArg==1 );

if( sqlite3_value_type(apArg[0])!=SQLITE_INTEGER ){

    sqlite3_result_error(ctx, "invalid argument", -1);

    返回;

}

pInt = (sqlite3_int64*)sqlite3_aggregate_context(ctx, sizeof(sqlite3_int64));

if( pInt ){

    *pInt += sqlite3_value_int64(apArg[0]);

}

}

*/**

*** sumint() 的 xInverse。*

****

*** 这与 xStep() 相反 - 减去参数的值*

*** 从当前上下文值返回。可以从*

*** 此函数不会分配任何超出缓冲区的资源*

*** 上下文已分配，并且没有可释放的资源。因此，此方法*

*** 已经无误地传递给 xStep()（因此必须是整数）。*

**/*

static void sumintInverse(

sqlite3_context *ctx,

int nArg,

sqlite3_value *apArg[]

){

sqlite3_int64 *pInt;

assert( sqlite3_value_type(apArg[0])==SQLITE_INTEGER );

pInt = (sqlite3_int64*)sqlite3_aggregate_context(ctx, sizeof(sqlite3_int64));

*pInt -= sqlite3_value_int64(apArg[0]);

}

*/**

*** sumint() 的 xFinal。*

****

*** 返回聚合窗口函数的当前值。因为*

*** 此实现不会分配任何超出 sqlite3_aggregate_context 返回的资源，该函数会自动释放*

*** 返回的值（此上下文已分配）且具有已传递的值*

*** 系统没有资源可供释放。因此，此方法*

*** 与 xValue() 相同。*

**/*

static void sumintFinal(sqlite3_context *ctx){

sqlite3_int64 res = 0;

sqlite3_int64 *pInt;

pInt = (sqlite3_int64*)sqlite3_aggregate_context(ctx, 0);

if( pInt ) res = *pInt;

sqlite3_result_int64(ctx, res);

}

*/**

*** sumint() 的 xValue。*

****

*** 返回聚合窗口函数的当前值。*

**/*

static void sumintValue(sqlite3_context *ctx){

sqlite3_int64 res = 0;

sqlite3_int64 *pInt;

pInt = (sqlite3_int64*)sqlite3_aggregate_context(ctx, 0);

if( pInt ) res = *pInt;

sqlite3_result_int64(ctx, res);

}

*/**

*** 使用数据库句柄 db 注册 sumint() 窗口聚合函数。*

**/*

int register_sumint(sqlite3 *db){

return sqlite3_create_window_function(db, "sumint", 1, SQLITE_UTF8, 0,

    sumintStep, sumintFinal, sumintValue, sumintInverse, 0

);

}

```sql

```

下面的示例使用了上述 C 代码实现的 sumint() 函数。对于每一行，窗口由前一行（如果有）、当前行和后一行（如果有）组成：

```sql
CREATE TABLE t3(x, y);
INSERT INTO t3 VALUES('a', 4),
                     ('b', 5),
                     ('c', 3),
                     ('d', 8),
                     ('e', 1);

*-- Assuming the database is populated using the above script, the* 
*-- following SELECT statement returns:*
*--* 
*--   x | sum_y*
--------------
*--   a | 9* 
*--   b | 12* 
*--   c | 16* 
*--   d | 12* 
*--   e | 9* 
*--* 
SELECT x, sumint(y) OVER (
  ORDER BY x ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
) AS sum_y
FROM t3 ORDER BY x;

```

在处理上述查询时，SQLite 按以下方式调用 sumint 回调函数：

1.  **xStep(4)** - 在当前窗口中添加 "4"。

1.  **xStep(5)** - 在当前窗口中添加 "5"。

1.  **xValue()** - 调用 xValue() 获取 (x='a') 行的 sumint() 的值。当前窗口包含值 4 和 5，因此结果为 9。

1.  **xStep(3)** - 在当前窗口中添加 "3"。

1.  **xValue()** - 调用 xValue() 获取 (x='b') 行的 sumint() 的值。当前窗口包含值 4、5 和 3，因此结果为 12。

1.  **xInverse(4)** - 从窗口中移除 "4"。

1.  **xStep(8)** - 在当前窗口中添加 "8"。窗口现在包含值 5、3 和 8。

1.  **xValue()** - 调用以获取 (x='c') 行的值。在这种情况下，为 16。

1.  **xInverse(5)** - 从窗口中移除值 "5"。

1.  **xStep(1)** - 向窗口中添加值 "1"。

1.  **xValue()** - 被调用以获取行（x='d'）的值。

1.  **xInverse(3)** - 从窗口中移除值 "3"。现在窗口只包含值 8 和 1。

1.  **xFinal()** - 被调用以回收任何分配的资源，并获取行（x='e'）的值。9\. .

如果用户在 SQLite 调用 xFinal() 前通过调用 sqlite3_reset() 或 sqlite3_finalize() 放弃了查询执行，则在 sqlite3_reset() 或 sqlite3_finalize() 调用内部会自动调用 xFinal()，以回收任何分配的资源，即使不需要值也是如此。在这种情况下，xFinal 实现返回的任何错误都将被静默丢弃。

# 6\. 历史

窗口函数支持首次添加到 SQLite 中的版本是 版本 3.25.0 (2018-09-15)。SQLite 开发人员使用 [PostgreSQL](http://www.postgresql.org) 窗口函数文档作为它们窗口函数应如何行为的主要参考。已运行许多测试用例来确保窗口函数在 SQLite 和 PostgreSQL 中运行方式一致。

在 SQLite 版本 3.28.0 (2019-04-16) 中，窗口函数支持已扩展以包括 EXCLUDE 子句、GROUPS 窗口类型、窗口链以及对 "<expr> PRECEDING" 和 "<expr> FOLLOWING" 边界的支持。
