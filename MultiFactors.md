---


---

<p>【米筐金工】多因子模型的步骤梳理</p>
<p>原创 2018-01-03 金尾巴 Ricequant</p>
<p>在量化交易中，<strong>多因子策略</strong>是一种常被提及且应用广泛的选股策略。我们会经常使用某种指标或者多种指标来对股票池进行筛选，这些用于选股的指标一般被称为因子。顾名思义，多因子模型是指使用多个因子，综合考量各因素而建立的选股模型，其假设股票收益率能被一组共同因子和个股特异因素所解释。</p>
<p>多因子模型的优点在于，<strong>它能通过有限共同因子来有效地筛选数量庞大的个股</strong>，在大幅度降低问题难度的同时，也通过合理预测做出了判断。本篇将对<strong>如何构建多因子模型做详细介绍</strong>，同时在各步骤附上相关帖子。</p>
<p><img src="https://mmbiz.qpic.cn/mmbiz_png/iaHVEbyZpr5ec7nk3aiaBfA2fxwrs7bCSOU7WlDxr7j0eibh4BKTk0Gt8l2icTm9D3gcrPyONIUib52N5RI4kfpLx9g/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" alt=""></p>
<p>（ 图1：多因子模型流程 ）</p>
<h2 id="一、数据预处理">一、数据预处理</h2>
<p>在构建多因子模型之前，我们首先要准备好待检验的原始因子池以及它们的数据，并根据要求对它们进行初步的整理。</p>
<h3 id="基础数据采集">1.1    基础数据采集</h3>
<p>作为建立模型最开始的第一步，确保使用数据的全面性和合理性是很重要的。我们<strong>首先需要归纳出不同风格的因子种类</strong>，再在各个风格大类下细分相关因子，并综合经济含义以及相关参数来确定因子的计算方法。</p>
<p>风格因子是指该种类因子具有一种独特的总体表现，根据Barra的定义可以分为9类，分别是Beta，动量，规模，盈利性，波动性，成长性，价值，杠杆率和流动性。每个大类因子里面还有细分的因子。除此之外，还有各种被探索出来的新因子，以期能更好的分析不同市场时期所展示的特征表现。</p>
<p>在Ricequant平台可以通过get_fundamentals拿到股票的财务数据，目前的提供的财务数据可以在财务数据文档中找到相应字段。我们的财务数据来源于国内最好的金融数据供应商之一的恒生聚源，从而极好地保证了策略输出结果的准确性。</p>

<table>
<thead>
<tr>
<th>大类因子</th>
<th>因子简称</th>
<th>因子解释</th>
</tr>
</thead>
<tbody>
<tr>
<td>估值因子</td>
<td>pe_ratio</td>
<td>市盈率</td>
</tr>
<tr>
<td></td>
<td>pb_ratio</td>
<td>市净率</td>
</tr>
<tr>
<td></td>
<td>pcf_ratio</td>
<td>市现率</td>
</tr>
<tr>
<td></td>
<td>ps_ratio</td>
<td>市销率</td>
</tr>
<tr>
<td></td>
<td>peg_ratio</td>
<td>市盈率相对盈利增长率</td>
</tr>
<tr>
<td></td>
<td>EBIT/EV</td>
<td>息税前利润/企业价值</td>
</tr>
<tr>
<td>波动率因子</td>
<td>idio_vol_FF</td>
<td>基于Fama的特质波动率</td>
</tr>
<tr>
<td></td>
<td>idio_vol_CAPM</td>
<td>基于CAPM的特质波动率</td>
</tr>
<tr>
<td></td>
<td>idio_vol_Carhart</td>
<td>基于Carhart的特质波动率</td>
</tr>
<tr>
<td></td>
<td>downside_idio_vol_FF</td>
<td>基于Fama的特质下行波动率</td>
</tr>
<tr>
<td>杠杆因子</td>
<td>debt_to_equity_ratio</td>
<td>负债权益比</td>
</tr>
<tr>
<td></td>
<td>debt_to_asset_ratio</td>
<td>资产负债比</td>
</tr>
<tr>
<td></td>
<td>tangible_asset_to_debt</td>
<td>有形资产负债比</td>
</tr>
<tr>
<td></td>
<td>current_ratio</td>
<td>现金比率</td>
</tr>
<tr>
<td></td>
<td>quick_ratio</td>
<td>流动比率</td>
</tr>
</tbody>
</table><p>表1：部分初始因子池示例</p>
<h3 id="离群值处理">1.2   离群值处理</h3>
<p>对数据进行标准化之前，我们需要先对离群值进行处理。因为过大或过小的数据可能会影响到分析结果，尤其是在做回归的时候，离群值会严重影响因子和收益率之间的相关性估计结果。</p>
<p>离群值的处理方法是将其调整至上下限，其中上下限由离群值判断的标准给出。离群值的判断标准有三种，分别为 MAD、 3σ、百分位法，主要思路是先界定上下限，再将超过界限的离群值调整至上下限。比较常用的是MAD法。</p>
<h3 id="数据标准化">1.3    数据标准化</h3>
<p>即使同属于一种风格因子，各个细分因子间的量级和单位也可能会有很大的差别。为了更好地对因子们进行比较和回归，我们需要对因子进行标准化处理。</p>
<p>标准化（standardization）在统计学中有一系列含义，一般使用z-score的方法。处理后的数据从有量纲转化为无量纲，从而使得数据更加集中，或者使得不同的指标能够进行比较和回归。</p>
<p>对因子进行标准化处理的方法主要有以下两种：</p>
<p><strong>1、对原始因子值进行标准化；</strong></p>
<p><strong>2、用因子的排序值进行标准化。</strong></p>
<p>实际上方法一更加常用，因为可以保留更多的因子分布信息，但是需要去掉极端值，否则会影响到回归结果。回归的方法一般使用z-score，将因子值的均值调整为0，标准差调整为1。</p>
<p>1.2 &amp; 1.3的详细方法介绍和相关代码可见帖子<a href="http://mp.weixin.qq.com/s?__biz=MzA4NTMyOTU3Ng==&amp;mid=2649582447&amp;idx=1&amp;sn=0f1746eb53b4741665bc3fd7bd75c853&amp;chksm=87c058a8b0b7d1be6350b4dff9e0253facc96019953b9c1d423d52000fe59312a44a4346ebb4&amp;scene=21#wechat_redirect">《数据预处理（上）之离群值处理、标准化》</a>。</p>
<p><img src="https://mmbiz.qpic.cn/mmbiz_png/iaHVEbyZpr5ec7nk3aiaBfA2fxwrs7bCSOehIenibTtwHLCtgkVjoYSQxaYa6nJNxiaKa2AjyIAzf1jYKGnGM2D2GQ/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" alt=""></p>
<p>图2：标准化处理后因子分布图</p>
<hr>
<h2 id="二、单因子检验">二、单因子检验</h2>
<p>我们在最开始采集数据时所初步构建的因子池，<strong>在逻辑上与收益率是有着一定经济意义上的联系</strong>，接下来我们需要对它们进行实证分析，筛选掉与收益率相关性不高的因子，从而得到真正有效的因子池。此部分可见帖子<a href="http://mp.weixin.qq.com/s?__biz=MzA4NTMyOTU3Ng==&amp;mid=2649582411&amp;idx=1&amp;sn=4a03bc7954b74ed31eb9256fd198ffbe&amp;chksm=87c0588cb0b7d19ad7cf9503dd20ce9ded4cffce36627fd941c73b7dad3bf27ad5b48fbef224&amp;scene=21#wechat_redirect">《波动率因子分析》</a>。</p>
<h3 id="特征分析">2.1    特征分析</h3>
<p>首先，初步分析因子之间的相关性，判断因子们的表现是否大致类似。</p>
<p>其次，用 pearson 或 spearman 方法计算因子的自相关系数，并观察因子的衰退速率是否有显著区别。</p>
<h3 id="中性化处理">2.2    中性化处理</h3>
<p>在使用这些因子进行选股时，有时会因为其它因子的影响，而导致选出来的股票具有一些我们不希望看到的偏向。比如说，市净率会与市值有很高的相关性，这时如果我们使用未进行市值中性化的市净率，选股的结果会比较集中。所以我们在使用这些因子之前，<strong>需要对它们是否对市值和行业有偏好进行检验和处理</strong>。</p>
<p>实际上中性化的含义不止对因子的中性化，详细和下文相关代码可以见<a href="http://mp.weixin.qq.com/s?__biz=MzA4NTMyOTU3Ng==&amp;mid=2649582511&amp;idx=1&amp;sn=a8428fa9591a43d9632d5f9c97d2157a&amp;chksm=87c05868b0b7d17e42053bc400158d895ed2104b3437cd128f407a68b995ff4a23ba3fb834ad&amp;scene=21#wechat_redirect">《数据预处理（下）之中性化》</a></p>
<h4 id="市值分析">2.2.1 市值分析</h4>
<p>为了确定市值因子是否影响待分析因子的暴露度，我们将全市场股票的市值取对数后由小至大分成5组宽度相等的市值区间，<strong>构造待分析因子的市值分布差异表</strong>。若因子暴露度表现出与市值有着明显相关性，则使用因子时需要对其进行市值中性化处理。</p>
<p>除了暴露度的市值分析，我们也可以对因子的IC值进行市值分析，查看在不同市值区间中因子的IC值是否有显著变化。</p>
<h4 id="行业分析">2.2.2 行业分析</h4>
<p>与市值分析类似，我们需要对因子暴露度和IC值做行业分析，并观察其行业分布结果。如果表现出明显行业分布差异，则使用该因子进行选股时，可以采取行业中性化处理。</p>
<p><img src="https://mmbiz.qpic.cn/mmbiz_png/iaHVEbyZpr5ec7nk3aiaBfA2fxwrs7bCSOCXKQPQDiaZoMglTicicThWpHWGtaKvNs47vbgVU3da7ljvqtq6DDibqUYw/640?wx_fmt=png&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" alt=""></p>
<p>图3：行业、市值中性化处理对比图</p>
<h3 id="回归法分析">2.3    回归法分析</h3>
<p><strong>回归法是最常用于检验因子有效性的方法</strong>，具体来说它是将T期因子的暴露度与T+1的股票收益率进行回归，所得的回归系数即为T期的因子收益率。回归模型中包含行业哑变量，若因子在前面的行业分析中发现与行业有明显相关性，则该模型能够排除行业差异影响。模型具体如下：
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>γ</mi><mi>i</mi><mrow><mi>T</mi><mo>+</mo><mn>1</mn></mrow></msubsup><mo>=</mo><msub><mo>∑</mo><mi>j</mi></msub><mrow><msubsup><mi>β</mi><mi>j</mi><mi>T</mi></msubsup><mo>∗</mo><mi>I</mi><mi>n</mi><mi>d</mi><mi>u</mi><mi>s</mi><mi>t</mi><mi>r</mi><msubsup><mi>y</mi><mrow><mi>j</mi><mo separator="true">,</mo><mi>i</mi></mrow><mi>T</mi></msubsup></mrow><mo>+</mo><msubsup><mi>β</mi><mi>F</mi><mi>T</mi></msubsup><mo>∗</mo><mi>F</mi><mi>a</mi><mi>c</mi><mi>t</mi><mi>o</mi><msubsup><mi>r</mi><mi>i</mi><mi>T</mi></msubsup><mo>+</mo><msubsup><mi>μ</mi><mi>i</mi><mi>T</mi></msubsup></mrow><annotation encoding="application/x-tex">
\gamma_i^{T+1}=\sum_j{\beta_j^T * Industry_{j,i}^T} + \beta_F^T * Factor_i^T + \mu_i^T
</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 1.05001em;"></span><span class="strut bottom" style="height: 2.46378em; vertical-align: -1.41378em;"></span><span class="base"><span class="mord"><span class="mord mathit" style="margin-right: 0.05556em;">γ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -2.43301em; margin-left: -0.05556em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span><span class="mbin mtight">+</span><span class="mord mathrm mtight">1</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.266995em;"></span></span></span></span></span><span class="mrel">=</span><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.05001em;"><span class="" style="top: -1.87233em; margin-left: 0em;"><span class="pstrut" style="height: 3.05em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.05724em;">j</span></span></span><span class="" style="top: -3.05em;"><span class="pstrut" style="height: 3.05em;"></span><span class=""><span class="mop op-symbol large-op">∑</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 1.41378em;"></span></span></span></span><span class="mord"><span class="mord"><span class="mord mathit" style="margin-right: 0.05278em;">β</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -2.453em; margin-left: -0.05278em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.05724em;">j</span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.383108em;"></span></span></span></span></span><span class="mbin">∗</span><span class="mord mathit" style="margin-right: 0.07847em;">I</span><span class="mord mathit">n</span><span class="mord mathit">d</span><span class="mord mathit">u</span><span class="mord mathit">s</span><span class="mord mathit">t</span><span class="mord mathit" style="margin-right: 0.02778em;">r</span><span class="mord"><span class="mord mathit" style="margin-right: 0.03588em;">y</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -2.453em; margin-left: -0.03588em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight" style="margin-right: 0.05724em;">j</span><span class="mpunct mtight">,</span><span class="mord mathit mtight">i</span></span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.383108em;"></span></span></span></span></span></span><span class="mbin">+</span><span class="mord"><span class="mord mathit" style="margin-right: 0.05278em;">β</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -2.453em; margin-left: -0.05278em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">F</span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.247em;"></span></span></span></span></span><span class="mbin">∗</span><span class="mord mathit" style="margin-right: 0.13889em;">F</span><span class="mord mathit">a</span><span class="mord mathit">c</span><span class="mord mathit">t</span><span class="mord mathit">o</span><span class="mord"><span class="mord mathit" style="margin-right: 0.02778em;">r</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -2.453em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.247em;"></span></span></span></span></span><span class="mbin">+</span><span class="mord"><span class="mord mathit">μ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -2.453em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.247em;"></span></span></span></span></span></span></span></span></span></span></p>
<ul>
<li><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>γ</mi><mi>i</mi><mrow><mi>T</mi><mo>+</mo><mn>1</mn></mrow></msubsup></mrow><annotation encoding="application/x-tex">\gamma_i^{T+1}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.881462em;"></span><span class="strut bottom" style="height: 1.15833em; vertical-align: -0.276864em;"></span><span class="base"><span class="mord"><span class="mord mathit" style="margin-right: 0.05556em;">γ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.881462em;"><span class="" style="top: -2.42314em; margin-left: -0.05556em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span><span class="" style="top: -3.10313em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span><span class="mbin mtight">+</span><span class="mord mathrm mtight">1</span></span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.276864em;"></span></span></span></span></span></span></span></span></span>：股票i在第T+1期的收益率</li>
<li><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>F</mi><mi>a</mi><mi>c</mi><mi>t</mi><mi>o</mi><msubsup><mi>r</mi><mi>i</mi><mi>T</mi></msubsup></mrow><annotation encoding="application/x-tex">Factor_i^T</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.841331em;"></span><span class="strut bottom" style="height: 1.09999em; vertical-align: -0.258664em;"></span><span class="base"><span class="mord mathit" style="margin-right: 0.13889em;">F</span><span class="mord mathit">a</span><span class="mord mathit">c</span><span class="mord mathit">t</span><span class="mord mathit">o</span><span class="mord"><span class="mord mathit" style="margin-right: 0.02778em;">r</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.841331em;"><span class="" style="top: -2.44134em; margin-left: -0.02778em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.258664em;"></span></span></span></span></span></span></span></span></span>：股票i在第T期因子Factor上的暴露度</li>
<li><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>I</mi><mi>n</mi><mi>d</mi><mi>u</mi><mi>s</mi><mi>t</mi><mi>r</mi><msubsup><mi>y</mi><mrow><mi>j</mi><mo separator="true">,</mo><mi>i</mi></mrow><mi>T</mi></msubsup></mrow><annotation encoding="application/x-tex">Industry_{j,i}^T</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.841331em;"></span><span class="strut bottom" style="height: 1.2361em; vertical-align: -0.394772em;"></span><span class="base"><span class="mord mathit" style="margin-right: 0.07847em;">I</span><span class="mord mathit">n</span><span class="mord mathit">d</span><span class="mord mathit">u</span><span class="mord mathit">s</span><span class="mord mathit">t</span><span class="mord mathit" style="margin-right: 0.02778em;">r</span><span class="mord"><span class="mord mathit" style="margin-right: 0.03588em;">y</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.841331em;"><span class="" style="top: -2.44134em; margin-left: -0.03588em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mtight"><span class="mord mathit mtight" style="margin-right: 0.05724em;">j</span><span class="mpunct mtight">,</span><span class="mord mathit mtight">i</span></span></span></span><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.394772em;"></span></span></span></span></span></span></span></span></span>：股票i在第j个行业因子上的暴露度(属于该行业则为1，否则为0)</li>
<li><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>β</mi><mi>j</mi><mi>T</mi></msubsup></mrow><annotation encoding="application/x-tex">\beta_j^T</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.841331em;"></span><span class="strut bottom" style="height: 1.2361em; vertical-align: -0.394772em;"></span><span class="base"><span class="mord"><span class="mord mathit" style="margin-right: 0.05278em;">β</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.841331em;"><span class="" style="top: -2.44134em; margin-left: -0.05278em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.05724em;">j</span></span></span><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.394772em;"></span></span></span></span></span></span></span></span></span>：第T期第j个行业因子的因子收益率，需回归拟合</li>
<li><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>β</mi><mi>F</mi><mi>T</mi></msubsup></mrow><annotation encoding="application/x-tex">\beta_F^T</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.841331em;"></span><span class="strut bottom" style="height: 1.11666em; vertical-align: -0.275331em;"></span><span class="base"><span class="mord"><span class="mord mathit" style="margin-right: 0.05278em;">β</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.841331em;"><span class="" style="top: -2.42467em; margin-left: -0.05278em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">F</span></span></span><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.275331em;"></span></span></span></span></span></span></span></span></span>：第T期因子Factor的因子收益率，需回归拟合</li>
<li><span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><msubsup><mi>μ</mi><mi>i</mi><mi>T</mi></msubsup></mrow><annotation encoding="application/x-tex">\mu_i^T</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.841331em;"></span><span class="strut bottom" style="height: 1.09999em; vertical-align: -0.258664em;"></span><span class="base"><span class="mord"><span class="mord mathit">μ</span><span class="msupsub"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.841331em;"><span class="" style="top: -2.44134em; margin-left: 0em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight">i</span></span></span><span class="" style="top: -3.063em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.258664em;"></span></span></span></span></span></span></span></span></span>：股票i在第T期的残差收益率</li>
</ul>
<p>在进行回归法分析中，我们需要对数据进一步处理。除了在对数据进行标准化及离群值处理，我们还<strong>需要对因子的缺失值进行填补，从而提升回归结果的可信度</strong>。同时，由于可能存在小盘股的影响以及回归的异方差性，我们采用加权最小二乘回归（WLS），权重为个股流通市值的平方根。</p>
<p>如果因为出现缺失值就将该个股删除，可能会导致不同因子回归的股票池差距较大，或者导致股票池大大缩水。对于缺失值的填补比较常用的方法是设为0、均值、上下数据、插值法，和算法拟合进行填充。</p>
<p>由此我们可以得到回归时间段中的因子收益率序列，以及相应的因子收益率t值序列。我们<strong>通过分析t值，能够判断出对应回归系数的显著性</strong>，从而得出该因子是否的确对下期的股票收益率有解释作用。</p>
<p>评价方法：</p>
<p><strong>a.    t值绝对值均值：判断显著性</strong></p>
<p><strong>b.    因子收益率大于0的占比：判断该因子对股票收益率的正向影响是否明显</strong></p>
<p><strong>c.    t值绝对值中大于2的占比：判断显著性是否稳定</strong></p>
<p><strong>d.    因子收益率零假设的t值：判断该因子的收益率序列是否显著不为零。</strong></p>
<p>（此部分内容的具体代码和分析将在下次帖子更新）</p>
<h3 id="ic法辅助分析">2.4    IC法辅助分析</h3>
<p>因子有效性是指因子是否可以获得持续、稳定的alpha收益。本部分主要使用IC分析及其衍生的指标对因子的有效性进行评估</p>
<blockquote>
<p>IC（信息系数）定义为每个时间截点上因子在各个股票的暴露度和股票下期收益的 pearson 或 spearman相关系数，IC值越高意味着该因子的暴露度与未来收益率存在较明显的相关关系。</p>
</blockquote>
<p><strong>第一步，进行IC统计分析</strong>。</p>
<p>为了观察各因子与收益率是否有明显相关性，我们将比较IC值序列的均值大小（因子显著性）、标准差（因子稳定性）、IR比率（因子有效性），以及累积曲线（随时间变化效果是否稳定）这几方面对因子进行定性评价。其中，IR（信息比）是指残差收益率的年化预测值与其年化波动率之比，这里我们将之简化定义为因子在测试期间内IC的均值与IC的标准差的比值。</p>
<p><strong>第二步，进行IC特征分析</strong>。</p>
<p>由于市场风格是会轮动的，IC值可能会切换正负号，所以在选择因子时会计算相关系数的正负比例，并选择比例高的方向。 <strong>作为假如同向显著比例占上风，则意味着该段时间内因子的风格延续性较强</strong>，可以使用动态权重来调整因子的权重；若状态切换比例占上风，对于因子的赋权应该使用静态权重。</p>
<p>衡量因子方向的指标一般有正相关显著比例、负相关显著比例、同向比例和状态切换比例。</p>
<p><strong>第三步，IC时间序列分析</strong></p>
<p>通过使用移动平均线，对各因子在一定时间内的趋势进行横向比较，同时参考当时的重大市场行情变化。</p>
<h3 id="分层回测">2.5    分层回测</h3>
<p>按照因子大小对股票排序，将股票池均分为N个组合，或者对每个行业内进行均分。个股权重一般选择等权，行业间权重一般与基准（例如沪深300）的行业配比相同，此时的组合为行业中性。</p>
<p>通过分组累计收益图，<strong>就可以简单的知道因子是否和收益率有着单调递增或递减的关系</strong>。回测结果有很多评价标准，例如年化收益率、夏普比率、信息比率、最大回撤等。</p>
<hr>
<h2 id="三、大类因子合成">三、大类因子合成</h2>
<p>在经过前面的分析之后，我们已经筛选出与收益率有着显著关系的因子池。然而此时的因子们仍然是由我们主观定义的，<strong>它们互相之间可能存在着很强的相关性</strong>。如果不作处理，投资组合将在同种因子上暴露过多的风险，并且<strong>多重共线性会造成多元线性回归的结果偏差</strong>。</p>
<p>此部分内容可见帖子<a href="http://mp.weixin.qq.com/s?__biz=MzA4NTMyOTU3Ng==&amp;mid=2649582534&amp;idx=1&amp;sn=908e73f0acbe32dc4e781cab9de6fab7&amp;chksm=87c05801b0b7d117cf00be11a8e9d990d3478eea0c2701b88abd598c7d42e9a22b8e81fdbdb0&amp;scene=21#wechat_redirect">《多因子权重优化方法比较》</a></p>
<h3 id="细分因子间相关性分析">3.1    细分因子间相关性分析</h3>
<p>因子相关性可由 pearson 和 spearman方法计算得出。除了普通的相关性分析之外，因子的IC值整体变化方向的表现对相关性也具有一定的说明性。</p>
<h3 id="同种因子下的细分因子合成">3.2    同种因子下的细分因子合成</h3>
<p>提取细分因子的有效信息，并进行合成的方式主要三种：<strong>等权细分因子、利用PCA对高相关性因子进行降维、利用逐步回归筛选细分因子</strong>。不同的因子适合不同的合成方法，一般而言PCA适用于具有较强相关性的细分因子，但所得合成因子的经济意义可能不明显。</p>
<h3 id="合成因子间相关性检验">3.3    合成因子间相关性检验</h3>
<p>在得到合成的大类因子之后，<strong>需要对它们进行相关性检验</strong>。由于此时的因子之间不再具有相似经济含义，如果出现明显相关性，则考虑对合成因子进行取舍，从而保证多因子模型不仅是经济含义方面或收益效果都能达到最优。</p>
<hr>
<h2 id="四、构造模型">四、构造模型</h2>
<p>经过一系列的筛选和分析后，现在已经得到最终的因子集。本篇在此部分以打分法为例，回归法的步骤将在之后的文章更新。在打分法中，我们将赋予各因子权重，以期符合选股预期或经济逻辑。此部分可参考<a href="http://mp.weixin.qq.com/s?__biz=MzA4NTMyOTU3Ng==&amp;mid=2649582534&amp;idx=1&amp;sn=908e73f0acbe32dc4e781cab9de6fab7&amp;chksm=87c05801b0b7d117cf00be11a8e9d990d3478eea0c2701b88abd598c7d42e9a22b8e81fdbdb0&amp;scene=21#wechat_redirect">《多因子权重优化方法比较》</a>。</p>
<h3 id="确定因子权重">4.1    确定因子权重</h3>
<p>确定权重的方法有四种：</p>
<ul>
<li>
<p>各因子等权处理。缺点是未考虑各因子的有效性和稳定性差异。</p>
</li>
<li>
<p>因子IC均值加权。此方法考虑到了因子有效性的差异，将在表现更显著的因子上分配更好的权重。</p>
</li>
<li>
<p>IR_IC法加权。此方法根据收益-风险这一基本准则，综合考虑到了因子有效性和稳定性。</p>
</li>
<li>
<p>最大化复合因子IR。通过最大化多因子模型的IR来获得各因子的最优权重，并利用求解构造最佳多因子模型。此处可使用普通的协方差矩阵或者Ledoit-Wolf压缩方法得到协方差矩阵。</p>
</li>
</ul>
<p>通常而言，方法四即使用压缩矩阵最大化复合IR的权重配置方式的选股结果表现最佳。</p>
<h3 id="个股打分并筛选">4.2    个股打分并筛选</h3>
<p>在最开始的数据预处理中，已将各因子暴露度标准化，故可以通过权重算出个股的分值。根据打分后的结果，通常是按照一个比例（例如前30%），或者一个分值门槛作为筛选标准，买入评分高的股票。</p>
<p>此时可以通过简单的分配权重完成多因子模型构建，个股间的权重分配一般是等权，或者是按照市值大小进行加权得出。</p>
<hr>
<h2 id="五、组合优化">五、组合优化</h2>
<p>我们已经得到了打分法会使用到的基础数据，但这样很可能会出现我们不希望的情况，例如风险过多暴露在某一行业，所以需要对模型进行优化。</p>
<h3 id="添加约束条件">5.1    添加约束条件</h3>
<p>如果单纯地采取等权买入，风险可能会过多地暴露在某一不被希望的方面。常见的约束条件如下：行业权重约束、因子暴露约束、个股上下限、收益目标、风险目标。<strong>其中最后两项一般用于回归法的多因子模型构建中</strong>。</p>
<h3 id="二次规划求解权重">5.2    二次规划求解权重</h3>
<p>一般二次规划问题可以表示成如下形式：
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mrow><msub><mi>min</mi><mi>H</mi></msub><mrow><mfrac><mrow><mn>1</mn></mrow><mrow><mn>2</mn></mrow></mfrac><mo>∗</mo><msup><mi>H</mi><mi>T</mi></msup><mo>∗</mo><mi>Q</mi><mo>∗</mo><mi>H</mi></mrow></mrow><mo>+</mo><mrow><msup><mi>H</mi><mi>T</mi></msup><mo>∗</mo><mi>c</mi></mrow></mrow><annotation encoding="application/x-tex">{\min_H{\frac{1}{2}*H^T*Q*H}} + {H^T * c}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 1.32144em;"></span><span class="strut bottom" style="height: 2.06577em; vertical-align: -0.744331em;"></span><span class="base"><span class="mord"><span class="mop op-limits"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 0.66786em;"><span class="" style="top: -2.05567em; margin-left: 0em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.08125em;">H</span></span></span><span class="" style="top: -2.7em;"><span class="pstrut" style="height: 2.7em;"></span><span class=""><span class="mop">min</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.744331em;"></span></span></span></span><span class="mord"><span class="mord"><span class="mopen nulldelimiter"></span><span class="mfrac"><span class="vlist-t vlist-t2"><span class="vlist-r"><span class="vlist" style="height: 1.32144em;"><span class="" style="top: -2.314em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord mathrm">2</span></span></span><span class="" style="top: -3.23em;"><span class="pstrut" style="height: 3em;"></span><span class="frac-line" style="border-bottom-width: 0.04em;"></span></span><span class="" style="top: -3.677em;"><span class="pstrut" style="height: 3em;"></span><span class="mord"><span class="mord mathrm">1</span></span></span></span><span class="vlist-s">​</span></span><span class="vlist-r"><span class="vlist" style="height: 0.686em;"></span></span></span></span><span class="mclose nulldelimiter"></span></span><span class="mbin">∗</span><span class="mord"><span class="mord mathit" style="margin-right: 0.08125em;">H</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span></span></span></span></span><span class="mbin">∗</span><span class="mord mathit">Q</span><span class="mbin">∗</span><span class="mord mathit" style="margin-right: 0.08125em;">H</span></span></span><span class="mbin">+</span><span class="mord"><span class="mord"><span class="mord mathit" style="margin-right: 0.08125em;">H</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span></span></span></span></span><span class="mbin">∗</span><span class="mord mathit">c</span></span></span></span></span></span></span>
<span class="katex--display"><span class="katex-display"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>s</mi><mi mathvariant="normal">.</mi><mi>t</mi><mi mathvariant="normal">.</mi><mtext>&nbsp;</mtext><mtext>&nbsp;</mtext><mtext>&nbsp;</mtext><msup><mi>A</mi><mi>T</mi></msup><mo>∗</mo><mi>H</mi><mo>≤</mo><mi>b</mi></mrow><annotation encoding="application/x-tex">s.t. \ \ \    A^T * H \le b</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 0.891331em;"></span><span class="strut bottom" style="height: 1.0273em; vertical-align: -0.13597em;"></span><span class="base"><span class="mord mathit">s</span><span class="mord mathrm">.</span><span class="mord mathit">t</span><span class="mord mathrm">.</span><span class="mord"><span class="mspace">&nbsp;</span><span class="mspace">&nbsp;</span><span class="mspace">&nbsp;</span><span class="mord mathit">A</span><span class="msupsub"><span class="vlist-t"><span class="vlist-r"><span class="vlist" style="height: 0.891331em;"><span class="" style="top: -3.113em; margin-right: 0.05em;"><span class="pstrut" style="height: 2.7em;"></span><span class="sizing reset-size6 size3 mtight"><span class="mord mathit mtight" style="margin-right: 0.13889em;">T</span></span></span></span></span></span></span></span><span class="mbin">∗</span><span class="mord mathit" style="margin-right: 0.08125em;">H</span><span class="mrel">≤</span><span class="mord mathit">b</span></span></span></span></span></span></p>
<p>其中：</p>
<pre><code>H：需要求解的目标向量
Q：为最优化问题的二次项系数的对称半正定矩阵
c：为与线性目标方程有关的系数向量
A：为约束等式与非等式的系数矩阵
b：为约束值的向量矩阵
</code></pre>
<p>二次与线性最优化的问题都可以通过一般二次规划最优化程序来解决。对于线性最优化问题，<strong>只要令Q = 0，则问题变成一个线性规划问题</strong>。</p>
<p>由此，我们已经得到了添加各种约束后的个股权重，可以由此建立相应的多因子模型。</p>
<hr>
<h2 id="结语">结语</h2>
<p>通过上文的五个步骤，我们建立了以打分法实现的多因子模型，<strong>而实际上大部分的工作量主要集中于确定有效因子这一步</strong>。多因子策略也可以配合卖空对应的股指期货进行套保。</p>
<p>我们有多种指标来衡量多因子模型的绩效结果，例如2.5中提到的最大回撤等。在我们米筐平台中提供了绩效分析入口，其展现了Brinson分析、风格分析、净值回归以及绩效指标的结果。我们可以通过此报告进一步分析模型的结果和表现，进行业绩归因。</p>
<p>注意，多因子模型的构建与测试时间有关，如其他模型一样，此模型需要定期进行验证检查，以达到理想效果。</p>
<p><em>参考文献：</em></p>
<pre><code>《东方证券_20150909_因子选股系列研究之二：低特质波动，高超额收益》

《华泰单因子测试之波动率类因子》

《东方证券_20150626_因子选股系列研究之一：单因子有效性检验》

《华泰多因子系列之一：华泰多因子模型体系初探》
</code></pre>

