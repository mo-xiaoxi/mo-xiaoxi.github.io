---
layout: titled
nocomment: true
title: 来人，赏~
subtitle: 送小西一本书，让他写出更棒的作品！
---

{% include follow.html %}

我就是小西！活跃的 GitHub 用户一枚，喜欢在业余时间捣腾些<a href="https://github.com/momomoxiaoxi" target="_blank">有意思的项目</a>。

从 2016 年 1 月 1 日起，打赏任意一本书，发送邮件至 <a href="mailto:momomomoxiaoxi@gmail.com">momomomoxiaoxi@gmail.com</a> 告知你的地址邮编收件人，就可以在春节那天收到我**手绘的明信片**一枚~ 大约有一半的人没给我发邮件，不知道是不是不想告诉我地址的关系，那就只能收不到明信片咯~

## 方法一：支付宝扫一扫



## 方法二：转账到我的支付宝账号

`momomomoxiaoxi@gmail.com`

好了，不多说了，我学习技术去了。

<script type="text/javascript">
    var loadJs = [['{{ site.url }}/js/echarts-all.js', function() {
        // init echarts
        var chart = echarts.init($('#reading-chart')[0]);
        chart.setOption({
            tooltip: {
                trigger: 'value'
            },
            legend: {
                data:['2014', '2015']
            },
            grid: {
                x: 40,
                x2: 40,
                y: 40
            },
            calculable: true,
            xAxis: [{
                type: 'category',
                data: ['1月', '2月', '3月', '4月', '5月', '6月',
                        '7月', '8月', '9月', '10月', '11月', '12月'],
                axisLine: {
                    show: false
                }
            }],
            yAxis: [{
                type: 'value',
                axisLine: {
                    show: false
                }
            }],
            series: [{
                name: '2014',
                type: 'bar',
                data: [5, 28, 11, 9, 16, 9, 16, 13, 12, 5, 9, 9, 7, 6],
                itemStyle: {
                    normal: {
                        color: '#22C3AA'
                    }
                },
                markPoint: {
                    data: [{
                        type: 'max', 
                        name: '最大值'
                    }, {
                        type: 'min',
                        name: '最小值'
                    }]
                },
                markLine: {
                    data: [{
                        type: 'average',
                        name: '平均值'
                    }]
                }
            }, {
                name: '2015',
                type: 'bar',
                data: [7, 9, 9, 7, 8, 7, 4, 1, 9, 2],
                itemStyle: {
                    normal: {
                        color: '#D0648A'
                    }
                },
                markPoint: {
                    data: [{
                        type: 'max', 
                        name: '最大值'
                    }, {
                        type: 'min',
                        name: '最小值'
                    }]
                },
                markLine: {
                    data: [{
                        type: 'average',
                        name: '平均值'
                    }]
                }
            }]
        });

        $(window).resize(chart.resize);
    }]];
</script>
