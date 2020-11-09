---
title: 标量函数 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的标量函数。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 05/27/2020
ms.date: 09/30/2020
ms.openlocfilehash: 2f1bb1e0b7db62893e1e9cda6c19f41198d12ecc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106202"
---
# <a name="scalar-function-types"></a>标量函数类型

## <a name="binary-functions"></a>二进制函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[binary_and()](binary-andfunction.md)|返回两个值的“按位与”运算的结果。|
|[binary_not()](binary-notfunction.md)|返回输入值的“按位求反”结果。|
|[binary_or()](binary-orfunction.md)|返回两个值的“按位或”运算的结果。|
|[binary_shift_left()](binary-shift-leftfunction.md)|返回针对一对数字的二进制左移运算：a << n。|
|[binary_shift_right()](binary-shift-rightfunction.md)|返回针对一对数字的二进制右移运算：a >> n。|
|[binary_xor()](binary-xorfunction.md)|返回两个值的“按位异或”运算的结果。|
|[bitset_count_ones()](bitset-count-onesfunction.md)|返回数字的二进制表示形式中的设置位数。|

## <a name="conversion-functions"></a>转换函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[tobool()](toboolfunction.md)|将输入转换为布尔值（带符号的 8 位）表示形式。|
|[todatetime()](todatetimefunction.md)|将输入转换为日期/时间标量。|
|[todouble()/toreal()](todoublefunction.md)|将输入转换为类型为 real 的值。 （todouble() 和 toreal() 是同义函数。）|
|[tostring()](tostringfunction.md)|将输入转换为字符串表示形式。|
|[totimespan()](totimespanfunction.md)|将输入转换为时间跨度标量。|

## <a name="datetimetimespan-functions"></a>日期时间/时间跨度函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[ago()](agofunction.md)|从当前 UTC 时钟时间减去给定时间跨度。|
|[datetime_add()](datetime-addfunction.md)|计算新的日期时间，方法是将指定的日期部分与指定数量相乘，然后加上指定的日期时间。|
|[datetime_part()](datetime-partfunction.md)|将所请求的日期部分提取为整数值。|
|[datetime_diff()](datetime-difffunction.md)|返回包含日期的年份的终点，偏移某个偏移量（如已提供）。|
|[dayofmonth()](dayofmonthfunction.md)|返回一个整数，表示给定月份的第几天。|
|[dayofweek()](dayofweekfunction.md)|返回自上个星期日以来的整数天数，用作时间跨度。|
|[dayofyear()](dayofyearfunction.md)|返回一个整数，表示给定年份的第几天。|
|[endofday()](endofdayfunction.md)|返回包含日期的一天的终点，偏移某个偏移量（如已提供）。|
|[endofmonth()](endofmonthfunction.md)|返回包含日期的月份的终点，偏移某个偏移量（如已提供）。|
|[endofweek()](endofweekfunction.md)|返回包含日期的周的终点，偏移某个偏移量（如已提供）。|
|[endofyear()](endofyearfunction.md)|返回包含日期的年份的终点，偏移某个偏移量（如已提供）。|
|[format_datetime()](format-datetimefunction.md)|基于格式模式参数设置某个日期时间参数的格式。|
|[format_timespan()](format-timespanfunction.md)|基于格式模式参数设置某个格式-时间跨度参数的格式。|
|[getmonth()](getmonthfunction.md)|从日期时间获取月份号 (1-12)。|
|[getyear()](getyearfunction.md)|返回日期时间参数的年份部分。|
|[hourofday()](hourofdayfunction.md)|返回一个整数，表示给定日期中的第几个小时。|
|[make_datetime()](make-datetimefunction.md)|根据指定的日期和时间创建一个日期时间标量值。|
|[make_timespan()](make-timespanfunction.md)|根据指定的时间段创建一个时间跨度标量值。|
|[monthofyear()](monthofyearfunction.md)|返回一个整数，表示给定年份的第几个月。|
|[now()](nowfunction.md)|返回当前的 UTC 时钟时间，偏移量取决于给定的时间跨度（可选）。|
|[startofday()](startofdayfunction.md)|返回包含日期的一天的起点，偏移某个偏移量（如已提供）。|
|[startofmonth()](startofmonthfunction.md)|返回包含日期的月份的起点，偏移某个偏移量（如已提供）。|
|[startofweek()](startofweekfunction.md)|返回包含日期的周的起点，偏移某个偏移量（如已提供）。|
|[startofyear()](startofyearfunction.md)|返回包含日期的年份的起点，偏移某个偏移量（如已提供）。|
|[todatetime()](todatetimefunction.md)|将输入转换为日期时间标量。|
|[totimespan()](totimespanfunction.md)|将输入转换为时间跨度标量。|
|[unixtime_microseconds_todatetime()](unixtime-microseconds-todatetimefunction.md)|将 unix-epoch 微秒转换为 UTC 日期/时间。|
|[unixtime_milliseconds_todatetime()](unixtime-milliseconds-todatetimefunction.md)|将 unix-epoch 毫秒转换为 UTC 日期/时间。|
|[unixtime_nanoseconds_todatetime()](unixtime-nanoseconds-todatetimefunction.md)|将 unix-epoch 纳秒转换为 UTC 日期/时间。|
|[unixtime_seconds_todatetime()](unixtime-seconds-todatetimefunction.md)|将 unix-epoch 秒转换为 UTC 日期/时间。|
|[weekofyear()](weekofyearfunction.md)|返回一个表示周数的整数。|


## <a name="dynamicarray-functions"></a>动态/数组函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[array_concat()](arrayconcatfunction.md)|将多个动态数组连接到单个数组。|
|[array_iif()](arrayifffunction.md)|按元素对数组应用 iif 函数。|
|[array_index_of()](arrayindexoffunction.md)|搜索指定项的数组，并返回其位置。|
|[array_length()](arraylengthfunction.md)|计算动态数组中的元素数。|
|[array_rotate_left()](array_rotate_leftfunction.md)|将动态数组中的值向左旋转。|
|[array_rotate_right()](array_rotate_rightfunction.md)|将动态数组中的值向右旋转。|
|[array_shift_left()](array_shift_leftfunction.md)|将动态数组中的值向左移动。|
|[array_shift_right()](array_shift_rightfunction.md)|将动态数组中的值向右移动。|
|[array_slice()](arrayslicefunction.md)|提取动态数组的切片。|
|[array_sort_asc()](arraysortascfunction.md)|按升序对数组集合进行排序。|
|[array_sort_desc()](arraysortdescfunction.md)|按降序对数组集合进行排序。|
|[array_split()](arraysplitfunction.md)|生成从输入数组拆分的一个或多个数组。|
|[bag_keys()](bagkeysfunction.md)|枚举 dynamic 属性包对象中的所有根键。|
|[bag_merge()](bag-merge-function.md)|将多个 dynamic 属性包合并成一个包含所有属性的 dynamic 属性包。|
|[bag_remove_keys()](bag-remove-keys-function.md)|从 dynamic 属性包中删除键和关联的值。|
|[pack()](packfunction.md)|基于名称和值的列表创建一个动态对象（属性包）。|
|[pack_all()](packallfunction.md)|基于表格表达式的所有列创建一个动态对象（属性包）。|
|[pack_array()](packarrayfunction.md)|将所有输入值打包为动态数组。|
|[repeat()](repeatfunction.md)|生成包含一系列相等值的动态数组。|
|[set_difference()](setdifferencefunction.md)|返回一个数组，其中包含在第一个数组中但不在其他数组中的所有非重复值的集合。|
|[set_has_element()](sethaselementfunction.md)|确定指定的数组中是否包含指定的元素。|
|[set_intersect()](setintersectfunction.md)|返回一个数组，其中包含所有数组中的所有非重复值的集合。|
|[set_union()](setunionfunction.md)|返回一个数组，其中包含任何已提供的数组中的所有非重复值的集合。|
|[treepath()](treepathfunction.md)|枚举可标识动态对象中所有遗留内容的路径表达式。|
|[zip()](zipfunction.md)|zip 函数接受任意数量的动态数组。 返回一个数组，其每个元素都是一个数组，后者包含同一索引的输入数组的元素。|

## <a name="window-scalar-functions"></a>窗口标量函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[next()](nextfunction.md)|对于序列化的行集，根据偏移量返回后一行中指定列的值。|
|[prev()](prevfunction.md)|对于序列化的行集，根据偏移量返回前一行中指定列的值。|
|[row_cumsum()](rowcumsumfunction.md)|计算列的累计和。|
|[row_number()](rownumberfunction.md)|返回某个行在序列化行集（从给定索引开始的或默认从 1 开始的连续编号）中的编号。|

## <a name="flow-control-functions"></a>流控制函数

|函数名称            |描述                                             |
|-------------------------|--------------------------------------------------------|
|[toscalar()](toscalarfunction.md)|返回所计算的表达式的标量常数值。|

## <a name="mathematical-functions"></a>数学函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[abs()](abs-function.md)|计算输入的绝对值。|
|[acos()](acosfunction.md)|返回其余弦为指定数字的角度（cos() 的反运算）。|
|[asin()](asinfunction.md)|返回其正弦为指定数字的角度（sin() 的反运算）。|
|[atan()](atanfunction.md)|返回其正切为指定数字的角度（tan() 的反运算）。|
|[atan2()](atan2function.md)|计算正 x 轴与从原点到点 (y, x) 的射线之间的角度（以弧度表示）。|
|[beta_cdf()](beta-cdffunction.md)|返回标准的累积 beta 分布函数。|
|[beta_inv()](beta-invfunction.md)|返回 beta 累积概率 beta 密度函数的反函数。|
|[beta_pdf()](beta-pdffunction.md)|返回概率密度 beta 函数。|
|[cos()](cosfunction.md)|返回余弦函数。|
|[cot()](cotfunction.md)|计算指定角度的三角函数余切（以弧度为单位）。|
|[degrees()](degreesfunction.md)|使用“度数 = (180/PI) * 弧度角度”这个公式，将以弧度表示的角度值转换为以度表示的值。|
|[exp()](exp-function.md)|x 的以 e 为底数的指数函数，表示 e 的 x 次幂：e^x。|
|[exp10()](exp10-function.md)|x 的以 10 为底数的指数函数，表示 10 的 x 次幂：10^x。|
|[exp2()](exp2-function.md)|x 的以 2 为底数的指数函数，表示 2 的 x 次幂：2^x。|
|[gamma()](gammafunction.md)|计算 gamma 函数。|
|[isfinite()](isfinitefunction.md)|返回输入是否为有限值（不是无限值或 NAN）。|
|[isinf()](isinffunction.md)|返回输入是否为无限值（正值或负值）。|
|[isnan()](isnanfunction.md)|返回输入是否为非数字 (NAN) 值。|
|[log()](log-function.md)|返回自然对数函数。|
|[log10()](log10-function.md)|返回常用（以 10 为底数）对数函数。|
|[log2()](log2-function.md)|返回以 2 为底数的对数函数。|
|[loggamma()](loggammafunction.md)|计算 gamma 函数绝对值的对数。|
|[not()](notfunction.md)|对其布尔参数的值取反。|
|[pi()](pifunction.md)|返回 Pi (π) 的常数值。|
|[pow()](powfunction.md)|返回幂运算的结果。|
|[radians()](radiansfunction.md)|使用“弧度 = (PI/180) * 角度”这个公式，将以度表示的角度值转换为以弧度表示的值。|
|[rand()](randfunction.md)|返回一个随机数。|
|[range()](rangefunction.md)|生成包含一系列等间距值的动态数组。|
|[round()](roundfunction.md)|返回舍入到指定精度的源。|
|[sign()](signfunction.md)|数值表达式的符号。|
|[sin()](sinfunction.md)|返回正弦函数。|
|[sqrt()](sqrtfunction.md)|返回平方根函数。|
|[tan()](tanfunction.md)|返回正切函数。|
|[welch_test()](welch-testfunction.md)|计算 Welch-test 函数的 p 值。|

## <a name="metadata-functions"></a>元数据函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[column_ifexists()](columnifexists.md)|采用列名作为字符串和默认值。 返回对列的引用（如果存在），否则返回默认值。|
|[current_cluster_endpoint()](current-cluster-endpoint-function.md)|返回正在运行查询的当前群集。|
|[current_database()](current-database-function.md)|返回作用域中的数据库的名称。|
|[current_principal()](current-principalfunction.md)|返回正在运行此查询的当前主体。|
|[current_principal_details()](current-principal-detailsfunction.md)|返回运行查询的主体的详细信息。|
|[current_principal_is_member_of()](current-principal-ismemberoffunction.md)|检查运行查询的当前主体的组成员身份或主体标识。|
|[cursor_after()](cursorafterfunction.md)|用于访问在游标的上一个值之后引入的记录。|
|[estimate_data_size()](estimate-data-sizefunction.md)|返回表格表达式的选定列的估计数据大小。|
|[extent_id()](extentidfunction.md)|返回标识当前记录所在的数据分片（“盘区”）的唯一标识符。|
|[extent_tags()](extenttagsfunction.md)|返回一个动态数组，其中包含当前记录所在的数据分片（“盘区”）的标记。|
|[ingestion_time()](ingestiontimefunction.md)|检索记录的 $IngestionTime 隐藏日期时间列，或返回 NULL。|

## <a name="rounding-functions"></a>舍入函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[bin()](binfunction.md)|将值向下舍入到给定 bin 大小的整数倍。|
|[bin_at()](binatfunction.md)|将值向下舍入为固定大小的“bin”，并控制 bin 的起点。 （另请参阅 bin 函数。）|
|[ceiling()](ceilingfunction.md)|计算大于或等于指定数值表达式的最小整数。|
|[floor()](floorfunction.md)|将值向下舍入到给定 bin 大小的整数倍。|

## <a name="conditional-functions"></a>条件函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[case()](casefunction.md)|计算谓词列表并返回其谓词满足条件的第一个结果表达式|
|[coalesce()](coalescefunction.md)|计算表达式列表，并返回第一个非 null（或非空，适用于字符串）表达式。|
|[iif()/iff()](iiffunction.md)|评估第一个参数（谓词）并返回第二个或第三个参数的值，具体取决于谓词评估结果为 true（第二个）还是为 false（第三个）。|
|[max_of()](max-offunction.md)|返回所评估的几个数值表达式的最大值。|
|[min_of()](min-offunction.md)|返回所评估的几个数值表达式的最小值。|

## <a name="series-element-wise-functions"></a>系列的按元素函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[series_add()](series-addfunction.md)|计算两个数值系列输入按元素相加的结果。|
|[series_divide()](series-dividefunction.md)|计算两个数值系列输入按元素相除的结果。|
|[series_equals()](series-equalsfunction.md)|计算两个数值系列输入的按元素等于 (`==`) 逻辑运算。|
|[series_greater()](series-greaterfunction.md)|计算两个数值系列输入的按元素大于(`>`) 逻辑运算。|
|[series_greater_equals()](series-greater-equalsfunction.md)|计算两个数值系列输入的按元素大于等于 (`>=`) 逻辑运算。|
|[series_less()](series-lessfunction.md)|计算两个数值系列输入的按元素小于 (`<`) 逻辑运算。|
|[series_less_equals()](series-less-equalsfunction.md)|计算两个数值系列输入的按元素小于等于 (`<=`) 逻辑运算。|
|[series_multiply()](series-multiplyfunction.md)|计算两个数值系列输入按元素相乘的结果。|
|[series_not_equals()](series-not-equalsfunction.md)|计算两个数值系列输入的按元素不等于 (`!=`) 逻辑运算。|
|[series_subtract()](series-subtractfunction.md)|计算两个数值系列输入按元素相减的结果。|

## <a name="series-processing-functions"></a>序列处理函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[series_decompose()](series-decomposefunction.md)|将系列分解为组件。|
|[series_decompose_anomalies()](series-decompose-anomaliesfunction.md)|根据系列分解情况查找系列中的异常。|
|[series_decompose_forecast()](series-decompose-forecastfunction.md)|根据系列分解情况进行预测。|
|[series_fill_backward()](series-fill-backwardfunction.md)|在系列中对缺失值执行后向填充内插。|
|[series_fill_const()](series-fill-constfunction.md)|用指定的常数值替换系列中缺失的值。|
|[series_fill_forward()](series-fill-forwardfunction.md)|在系列中对缺失值执行前向填充内插。|
|[series_fill_linear()](series-fill-linearfunction.md)|在系列中对缺失值执行线性内插。|
|[series_fft()](series-fft-function.md)|对某个系列应用快速傅立叶变换 (FFT)。|
|[series_fir()](series-firfunction.md)|对系列应用有限脉冲响应滤波器。|
|[series_fit_2lines()](series-fit-2linesfunction.md)|对系列应用两段线性回归，返回多个列。|
|[series_fit_2lines_dynamic()](series-fit-2lines-dynamicfunction.md)|对系列应用两段线性回归，返回动态对象。|
|[series_fit_line()](series-fit-linefunction.md)|对系列应用线性回归，返回多个列。|
|[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)|对系列应用线性回归，返回动态对象。|
|[series_fit_poly()](series-fit-poly-function.md)|对系列应用多项式回归，返回多个列。|
|[series_ifft()](series-ifft-function.md)|对系列应用快速傅立叶逆变换 (IFFT)。|
|[series_iir()](series-iirfunction.md)|对系列应用无限脉冲响应滤波器。|
|[series_outliers()](series-outliersfunction.md)|对系列中的异常点进行评分。|
|[series_pearson_correlation()](series-pearson-correlationfunction.md)|计算两个系列的皮尔逊相关系数。|
|[series_periods_detect()](series-periods-detectfunction.md)|找出一个时序中最重要的周期。|
|[series_periods_validate()](series-periods-validatefunction.md)|检查时序是否包含给定长度的定期模式。|
|[series_seasonal()](series-seasonalfunction.md)|查找序列的季节性组件。|
|[series_stats()](series-statsfunction.md)|在多个列中返回系列的统计信息。|
|[series_stats_dynamic()](series-stats-dynamicfunction.md)|在动态对象中返回系列的统计信息。|

## <a name="string-functions"></a>字符串函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[base64_encode_tostring()](base64_encode_tostringfunction.md)|将字符串编码为 base64 字符串。|
|[base64_decode_tostring()](base64_decode_tostringfunction.md)|将 base64 字符串解码为 UTF-8 字符串。|
|[base64_decode_toarray()](base64_decode_toarrayfunction.md)|将 base64 字符串解码为长值的数组。|
|[countof()](countoffunction.md)|计算字符串中子字符串的出现次数。 纯字符串匹配项可能会重叠；正则表达式匹配项不会重叠。|
|[extract()](extractfunction.md)|从文本字符串中获取正则表达式的匹配项。|
|[extract_all()](extractallfunction.md)|从文本字符串中获取正则表达式的所有匹配项。|
|[extractjson()](extractjsonfunction.md)|使用路径表达式获取 JSON 文本外指定的元素。|
|[indexof()](indexoffunction.md)|函数会报告输入字符串中指定的字符串第一次出现时的索引（从零开始）。|
|[isempty()](isemptyfunction.md)|如果参数为空字符串或 null，则返回 true。|
|[isnotempty()](isnotemptyfunction.md)|如果参数不为空字符串或 null，则返回 true。|
|[isnotnull()](isnotnullfunction.md)|如果参数不为 null，则返回 true。|
|[isnull()](isnullfunction.md)|计算其唯一参数并返回一个布尔值，指示该参数的计算结果是否为 null 值。|
|[parse_command_line()](parse-command-line.md)|分析 Unicode 命令行字符串并返回命令行参数的数组。|
|[parse_csv()](parsecsvfunction.md)|拆分表示逗号分隔值的给定字符串，并返回包含这些值的字符串数组。|
|[parse_ipv4()](parse-ipv4function.md)|将输入转换为 long（带符号的 64 位）数字表示形式。|
|[parse_ipv4_mask()](parse-ipv4-maskfunction.md)|将输入字符串和 IP 前缀掩码转换为 long（带符号的 64 位）数字表示形式。|
|[parse_ipv6()](parse-ipv6function.md)|将 IPv6 或 IPv4 字符串转换为规范的 IPv6 字符串表示形式。|
|[parse_ipv6_mask()](parse-ipv6-maskfunction.md)|将 IPv6 或 IPv4 字符串和网络掩码转换为规范的 IPv6 字符串表示形式。|
|[parse_json()](parsejsonfunction.md)|将字符串解释为 JSON 值并返回动态值。|
|[parse_url()](parseurlfunction.md)|分析绝对 URL 字符串并返回一个动态对象，其中包含 URL 的所有部分。|
|[parse_urlquery()](parseurlqueryfunction.md)|分析 URL 查询字符串并返回一个动态对象，其中包含查询参数。|
|[parse_version()](parse-versionfunction.md)|将版本的输入字符串表示形式转换为可比较的十进制数字。|
|[replace()](replacefunction.md)|将所有正则表达式匹配项替换为其他字符串。|
|[reverse()](reversefunction.md)|此函数对输入字符串取反。|
|[split()](splitfunction.md)|根据给定分隔符拆分给定字符串，并返回具有所包含子字符串的字符串数组。|
|[strcat()](strcatfunction.md)|将 1 到 64 个参数连接起来。|
|[strcat_delim()](strcat-delimfunction.md)|将 2 到 64 个参数连接起来（分隔符作为第一个参数提供）。|
|[strcmp()](strcmpfunction.md)|比较两个字符串。|
|[strlen()](strlenfunction.md)|返回输入字符串的长度（以字符为单位）。|
|[strrep()](strrepfunction.md)|按提供的次数（默认值为 1）重复给定的字符串。|
|[substring()](substringfunction.md)|从源字符串的某个索引开始到字符串结尾，提取子字符串。|
|[toupper()](toupperfunction.md)|将字符串转换为大写。|
|[translate()](translatefunction.md)|将一组字符（“searchList”）替换为给定字符串中的另一组字符（“replacementList”）。|
|[trim()](trimfunction.md)|删除指定的正则表达式的所有前导匹配项和尾随匹配项。|
|[trim_end()](trimendfunction.md)|删除指定的正则表达式的尾随匹配项。|
|[trim_start()](trimstartfunction.md)|删除指定的正则表达式的前导匹配项。|
|[url_decode()](urldecodefunction.md)|此函数将编码的 URL 转换为常规 URL 表示形式。|
|[url_encode()](urlencodefunction.md)|此函数将输入 URL 的字符转换为可通过 Internet 传输的格式。|

## <a name="ipv4ipv6-functions"></a>IPv4/IPv6 函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[ipv4_compare()](ipv4-comparefunction.md)|比较两个 IPv4 字符串。|
|[ipv4_is_match()](ipv4-is-matchfunction.md)|匹配两个 IPv4 字符串。|
|[parse_ipv4()](parse-ipv4function.md)|将输入字符串转换为 long（带符号的 64 位）数字表示形式。|
|[parse_ipv4_mask()](parse-ipv4-maskfunction.md)|将输入字符串和 IP 前缀掩码转换为 long（带符号的 64 位）数字表示形式。|
|[ipv6_compare()](ipv6-comparefunction.md)|对两个 IPv4 或 IPv6 字符串进行比较。|
|[ipv6_is_match()](ipv6-is-matchfunction.md)|对两个 IPv4 或 IPv6 字符串进行匹配。|
|[parse_ipv6()](parse-ipv6function.md)|将 IPv6 或 IPv4 字符串转换为规范的 IPv6 字符串表示形式。|
|[parse_ipv6_mask()](parse-ipv6-maskfunction.md)|将 IPv6 或 IPv4 字符串和网络掩码转换为规范的 IPv6 字符串表示形式。|
|[format_ipv4()](format-ipv4-function.md)|使用网络掩码分析输入，并返回表示 IPv4 地址的字符串。|
|[format_ipv4_mask()](format-ipv4-mask-function.md)|使用网络掩码分析输入，并以 CIDR 表示法返回表示 IPv4 地址的字符串。|

## <a name="type-functions"></a>类型函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[gettype()](gettypefunction.md)|返回其单个参数的运行时类型。|

## <a name="scalar-aggregation-functions"></a>标量聚合函数

|函数名称     |描述                                          |
|-------------------------|--------------------------------------------------------|
|[dcount_hll()](dcount-hllfunction.md)|根据 hll 结果（由 hll 或 hll-merge 生成的）计算 dcount。|
|[hll_merge()](hllmergefunction.md)|合并 hll 结果（聚合版 hll-merge() 的标量版本）。|
|[percentile_tdigest()](percentile-tdigestfunction.md)|根据 tdigest 结果（由 tdigest 或 merge-tdigests 生成的）计算百分比结果。|
|[percentrank_tdigest()](percentrank-tdigestfunction.md)|计算数据集中某个值的百分比排名。|
|[rank_tdigest()](rank-tdigest.md)|计算集中某个值的相对排名。|
|[tdigest_merge()](tdigest-mergefunction.md)|合并 tdigest 结果（聚合版 tdigest-merge() 的标量版本）。|

## <a name="geospatial-functions"></a>地理空间函数

|函数名称|描述|
|--------------------------------------------------------------------------|--------------------------------------------------------|
|[geo_distance_2points()](geo-distance-2points-function.md)|计算地球上两个地理空间坐标之间的最短距离。|
|[geo_distance_point_to_line()](geo-distance-point-to-line-function.md)|计算地球上坐标和直线之间的最短距离。|
|[geo_point_in_circle()](geo-point-in-circle-function.md)|计算地理空间坐标是否在地球上的某个圆圈范围内。|
|[geo_point_in_polygon()](geo-point-in-polygon-function.md)|计算地理空间坐标是否在地球上的某个多边形或多多边形内。|
|[geo_point_to_geohash()](geo-point-to-geohash-function.md)|计算地理位置的 Geohash 字符串值。|
|[geo_geohash_to_central_point()](geo-geohash-to-central-point-function.md)|计算表示 Geohash 矩形区中心的地理空间坐标。|
|[geo_point_to_s2cell()](geo-point-to-s2cell-function.md)|计算地理位置的 S2 单元格字符串值。|
|[geo_s2cell_to_central_point()](geo-s2cell-to-central-point-function.md)|计算表示 S2 单元格中心的地理空间坐标。|
|[geo_polygon_to_s2cells()](geo-polygon-to-s2cells-function.md)|计算覆盖地球上的多边形或多多边形的 S2 单元标记。 有用的地理空间联接工具。|
|[geo_line_densify()](geo-line-densify-function.md)|通过添加中间点将平面线边缘转换为测地线。|
|[geo_polygon_densify()](geo-polygon-densify-function.md)|通过添加中间点将多边形或多边形集合的平面边缘转换为测地线。|

## <a name="hash-functions"></a>哈希函数

|函数名称|描述|
|--------------------------------------------------------------------------|--------------------------------------------------------|
|[hash()](hashfunction.md)|返回输入值的哈希值。|
|[hash_combine()](hash_combinefunction.md)|将两个或更多个哈希值组合起来。|
|[hash_many()](hash_manyfunction.md)|返回多个值的组合哈希值。|
|[hash_md5()](md5hashfunction.md)|返回输入值的 MD5 哈希值。|
|[hash_sha256()](sha256hashfunction.md)|返回输入值的 SHA256 哈希值。|
