---
title: 小爱课程表~
excerpt: MIUI12以上的小米自带课程表
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 前言
本文将从使用和实现两个方面来介绍小爱课程表~

**注意：**你的手机系统必须在MIUI12及以上。

## 使用
1. 打开小爱语音助手（从下边搜索栏或者语音说小爱同学都能打开）。
2. 说：打开课程表。
3. 点最右边标签，选择学校，选择我编写的版本（OSSA王跃霖）。
4. 找到教务网站导入（需要连接校园网），点进去。
5. 输入账号密码，点击信息查询标签，再点击学生课表查询标签。
6. 点击下方一键导入。
7. 导入成功后，选择当前周数，改成对应周数。

## 实现
1. 下载[AISchdule DevTools](https://cnbj1.fds.api.xiaomi.com/aife/AISchedule/AISchedule.zip)压缩包，解压到本地。
2. 下载Chrome，打开链接 chrome://extensions/ ，打开开发者模式，导入AISchedule DevTools文件夹。
3. 在Chrome打开一个新的Tab，打开、登陆自己的教务系统，进入课程页面。
4. 在网页内右键，选「检查」，打开Chrome开发者工具，会有新增的AISchedule标签，进入后请跟随新手引导，创建「适配项目」。**注意：**学校url请填写官方url，不要填写ip或者代理！
5. 当你点击provider/parser函数的代码区时，会跳转到sources中，在此你可以编辑scheduleHtmlProvider函数、scheduleHtmlParser函数，并打断点debug。
6. 你可以参考注释与示例函数，编写两个适配函数。按ctrl+s/cmd+s保存，然后在网页中右键，选「AISchedule DevTools 运行」，Console中会打印出运行结果，两个函数的输出会分别在新窗口中显示，以便你边参考边修改函数。scheduleHtmlParser函数的输出须符合以下数据结构。**注意：**代码中无需引用任何模块，不要带有“require”这样的字符
7. 经过调试，如果在console中看到「All run successfully」，然后你可以上传到手机端，进行E2E自测（端到端自测）。
8. 在手机端打开课程表的教务导入功能，搜索学校，选择自己提交的适配，你可亲自体验，验证可用性。如果你觉得没问题，请点击反馈按钮「完美」，至此，你的适配已完成，状态为审核中。
9. 工具中会保存你的历史适配记录和适配学校的状态，如果你的适配项目发布了，会为很多同学提供便利哦~

以上来自[小爱课程表 开发者工具 使用文档-本校模式](https://ldtu0m3md0.feishu.cn/docs/doccnhZPl8KnswEthRXUz8ivnhb#)官方提供的文档，下边是关于代码的内容~

本文写于20年09月11日，此期间因为疫情原因，学校分年级上课，各个年级上课时间和休息时间都不一样，所以我就let了四个年级的上下课时间表，心算的各个年级上下课时间，可能存在时间错误的问题，如果你的年级存在问题请联系我。

从官方文档知道，我们只需要编辑两个文件：
**scheduleHtmlProvider.js**
**scheduleHtmlPasrse.js**
第一个文件适配得较好，用来截取HTML片段，而你截取的片段将弹出窗口以页面的形式展现，如果你发现弹出窗口中已经包含了课程表，就可以开始编辑第二个文件了。

第二个文件是关键，我们需要通过js拿到我们想要的数据，然后对数据进行操作，这里以方正举例：

```javascript
function scheduleHtmlParser(html) {
    //除函数名外都可编辑
    //传入的参数为上一步函数获取到的html
    //可使用正则匹配
    //可使用解析dom匹配，工具内置了$，跟jquery使用方法一样，直接用就可以了，参考：https://juejin.im/post/5ea131f76fb9a03c8122d6b9
    //以下为示例，您可以完全重写或在此基础上更改

    let result = []
    let bbb = $('#table1 .timetable_con')
    let wyl_id = $('.timetable_title h6')[1].children[0].data.split('：')[1]
    for (let u = 0; u < bbb.length; u++) {
        let re = { sections: [], weeks: [] }
        let aaa = $(bbb[u]).find('span')
        let week = $(bbb[u]).parent('td')[0].attribs.id
        if (week) {
            re.day = week.split('-')[0]
        }
        for (let i = 0; i < aaa.length; i++) {

            if (aaa[i].attribs.title == '上课地点') {

                for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {
                    re.position = $(aaa[i]).next()[0].children[j].data
                }
            }
            if (aaa[i].attribs.title == '节/周') {

                for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {

                    let lesson = $(aaa[i]).next()[0].children[j].data
                    for (let a = Number(lesson.split(')')[0].split('(')[1].split('-')[0]); a < Number(lesson.split(')')[0].split('(')[1].split('-')[1].split('节')[0]) + 1; a++) {

                        re.sections.push({ section: a })
                    }
                    for (let a = Number(lesson.split(')')[1].split('-')[0]); a < Number(lesson.split(')')[1].split('-')[1].split('周')[0]) + 1; a++) {

                        re.weeks.push(a)
                    }
                }
            }

            if (aaa[i].attribs.title == '教师') {

                for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {
                    re.teacher = $(aaa[i]).next()[0].children[j].data
                }
            }

            if (aaa[i].attribs.class == 'title') {

                for (let j = 0; j < $(aaa[i]).children()[0].children.length; j++) {
                    re.name = $(aaa[i]).children()[0].children[j].data

                }
            }

        }
        result.push(re)
    }
    console.log(result)

    return {courseInfos: result}
}
```

从代码知道，这里的代码按jQuery的语法去写即可，正常情况下该代码可以正常读取到方正的课程表，只不过没有时间，我们需要的是添加一个时间，这里以我校举例，我校因为疫情原因分年级上课，各个年级上课时间和休息时间都不一样，所以我就let了四个年级的上下课时间表，然后通过js拿到了学号的text，对学号的1、2位进行判断可知该学生所在的年级，let一个列表用来存时间，然后将此二位数字当做判断条件进行赋值，再在最后的return中加入**sectionTimes: wyl_sectionTime**即可，代码如下：

```javascript
function scheduleHtmlParser(html) {
function scheduleHtmlParser(html) {
    //除函数名外都可编辑
    //传入的参数为上一步函数获取到的html
    //可使用正则匹配
    //可使用解析dom匹配，工具内置了$，跟jquery使用方法一样，直接用就可以了，参考：https://juejin.im/post/5ea131f76fb9a03c8122d6b9
    //以下为示例，您可以完全重写或在此基础上更改
    // //大一
    // const wyl_startTime1 = ["08:00", "08:50", "09:45", "10:35", "13:00", "13:50", "14:45", "15:35", "18:30", "19:20"]
    // const wyl_endTime1 = ["08:45", "09:35", "10:30", "11:20", "13:45", "14:35", "15:30", "16:20", "19:15", "20:05"]

    // //大二
    // const wyl_startTime2 = ["08:10", "09:00", "10:00", "10:50", "13:10", "14:00", "15:00", "15:50", "18:30", "19:20"]
    // const wyl_endTime2 = ["08:55", "09:45", "10:45", "11:35", "13:55", "14:45", "15:45", "16:35", "19:15", "20:05"]

    // //大三
    // const wyl_startTime3 = ["08:20", "09:10", "10:15", "11:05", "13:20", "14:10", "15:15", "16:05", "18:30", "19:20"]
    // const wyl_endTime3 = ["09:05", "09:55", "11:00", "11:50", "14:05", "14:55", "16:00", "16:50", "19:15", "20:05"]

    // //大四
    // const wyl_startTime4 = ["08:30", "09:20", "10:30", "11:20", "13:30", "14:20", "15:30", "16:20", "18:30", "19:20"]
    // const wyl_endTime4 = ["09:15", "10:05", "11:15", "12:05", "14:15", "15:05", "16:15", "17:05", "19:15", "20:05"]

    //算的头疼，没觉得code能力变强，倒是觉得心算能力增强了

    const wyl_startTime = ["08:20", "09:15", "10:20", "11:15", "13:30", "14:25", "15:30", "16:25", "18:10", "19:05"]
    const wyl_endTime = ["09:05", "10:00", "11:05", "12:00", "14:15", "15:10", "16:15", "17:10", "18:55", "19:50"]

    let result = []
    let bbb = $('#table1 .timetable_con')

    //学号头两位，取消分年级上课以后这个就没用了- -
    //let wyl_id = $('.timetable_title h6')[1].children[0].data.split('：')[1]

    for (let u = 0; u < bbb.length; u++) {
        let re = { sections: [], weeks: [] }
        let aaa = $(bbb[u]).find('span')
        let week = $(bbb[u]).parent('td')[0].attribs.id

        if (week) {
            re.day = week.split('-')[0]
        }
        for (let i = 0; i < aaa.length; i++) {

            if (aaa[i].attribs.title == '上课地点') {

                for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {
                    re.position = $(aaa[i]).next()[0].children[j].data
                }
            }
            if (aaa[i].attribs.title == '节/周') {

                // for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {

                //     let lesson = $(aaa[i]).next()[0].children[j].data

                //     for (let a = Number(lesson.split(')')[0].split('(')[1].split('-')[0]); a < Number(lesson.split(')')[0].split('(')[1].split('-')[1].split('节')[0]) + 1; a++) {

                //         re.sections.push({ section: a })
                //     }
                //     //如果这个是空的，就不再到下边进行计算
                //     //这里需要特判一下周六的形式政策

                //     let wylt0 = lesson.split(')')[1].split(',')[0]
                //     let wylt1 = lesson.split(')')[1].split(',')[1]

                //     if (wylt1 == undefined) {
                //         let wylfir = Number(lesson.split(')')[1].split('-')[1].split('周')[0]);
                //         console.log("look!")

                //         console.log(lesson.split(')')[1])
                //         for (let a = Number(lesson.split(')')[1].split('-')[0]); a < wylfir + 1; a++) {
                //             re.weeks.push(a)
                //         }
                //     }
                //     if (lesson[8] == '周' && lesson[9] == ',') {
                //         re.weeks.push(Number(lesson.split(')')[1].split('周,')[0]))
                //         re.weeks.push(Number(lesson.split(')')[1].split('周,')[1].split('周')[0]))
                //     }
                //     if (lesson[9] == '周' && lesson[10] == ',') {
                //         re.weeks.push(Number(lesson.split(')')[1].split('周,')[0]))
                //         re.weeks.push(Number(lesson.split(')')[1].split('周,')[1].split('周')[0]))
                //     }
                //     if (lesson.split(')')[1].split('-')[1] == null) continue;
                // }

                for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {
                    let lesson = $(aaa[i]).next()[0].children[j].data
                    for (let a = Number(lesson.split(')')[0].split('(')[1].split('-')[0]); a < Number(lesson.split(')')[0].split('(')[1].split('-')[1].split('节')[0]) + 1; a++) {
                        re.sections.push({ section: a })
                    }
                    //
                    let wylt0 = lesson.split(')')[1].split(',')[0]
                    let wylt1 = lesson.split(')')[1].split(',')[1]

                    // console.log("look0!")
                    // console.log(wylt0)
                    // console.log(wylt0.length)

                    //>=4代表把单周扔出去
                    if (wylt0.length >= 4) {
                        let wylfir = Number(wylt0.split('-')[1].split('周')[0]);
                        // console.log("look1!")

                        // console.log(wylfir)
                        for (let a = Number(wylt0.split('-')[0]); a < wylfir + 1; a++) {
                            re.weeks.push(a)
                        }
                    }

                    //处理第二个连贯周
                    if (wylt1 !== undefined && wylt1.length >= 4) {
                        let wylfir = Number(wylt1.split('-')[1].split('周')[0]);
                        // console.log("look2!")
                        // console.log(wylt1.split('-')[0]);
                        // console.log(wylfir)
                        for (let a = Number(wylt1.split('-')[0]); a < wylfir + 1; a++) {
                            re.weeks.push(a)
                        }
                    }

                    //处理形势与政策
                    if (wylt1 !== undefined && wylt1.indexOf('-') === -1 && wylt0.indexOf('-') === -1) {
                        re.weeks.push(wylt0[0])
                        re.weeks.push(wylt1[0])
                    }

                    //处理单周
                    if (wylt1 === undefined && wylt0.indexOf('-') === -1) {
                        re.weeks.push(wylt0.split('周')[0])
                    }


                    // for (let a = Number(lesson.split(')')[1].split('-')[0]); a < Number(lesson.split(')')[1].split('-')[1].split('周')[0]) + 1; a++) {

                    //     re.weeks.push(a)
                    // }
                }


            }

            if (aaa[i].attribs.title == '教师') {

                for (let j = 0; j < $(aaa[i]).next()[0].children.length; j++) {
                    re.teacher = $(aaa[i]).next()[0].children[j].data
                }
            }

            if (aaa[i].attribs.class == 'title') {

                for (let j = 0; j < $(aaa[i]).children()[0].children.length; j++) {
                    re.name = $(aaa[i]).children()[0].children[j].data

                }
            }

        }
        result.push(re)
    }
    console.log(result)

    let wyl_sectionTime = [{
            "section": 1,
            "startTime": wyl_startTime[0],
            "endTime": wyl_endTime[0]
        },
        {
            "section": 2,
            "startTime": wyl_startTime[1],
            "endTime": wyl_endTime[1]
        },
        {
            "section": 3,
            "startTime": wyl_startTime[2],
            "endTime": wyl_endTime[2]
        },
        {
            "section": 4,
            "startTime": wyl_startTime[3],
            "endTime": wyl_endTime[3]
        },
        {
            "section": 5,
            "startTime": wyl_startTime[4],
            "endTime": wyl_endTime[4]
        },
        {
            "section": 6,
            "startTime": wyl_startTime[5],
            "endTime": wyl_endTime[5]
        },
        {
            "section": 7,
            "startTime": wyl_startTime[6],
            "endTime": wyl_endTime[6]
        },
        {
            "section": 8,
            "startTime": wyl_startTime[7],
            "endTime": wyl_endTime[7]
        },
        {
            "section": 9,
            "startTime": wyl_startTime[8],
            "endTime": wyl_endTime[8]
        },
        {
            "section": 10,
            "startTime": wyl_startTime[9],
            "endTime": wyl_endTime[9]
        }
    ];
    // if (wyl_id[0] == 1) {
    //     if (wyl_id[1] == 7) {
    //         wyl_sectionTime = [{
    //                 "section": 1,
    //                 "startTime": wyl_startTime4[0],
    //                 "endTime": wyl_endTime4[0]
    //             },
    //             {
    //                 "section": 2,
    //                 "startTime": wyl_startTime4[1],
    //                 "endTime": wyl_endTime4[1]
    //             },
    //             {
    //                 "section": 3,
    //                 "startTime": wyl_startTime4[2],
    //                 "endTime": wyl_endTime4[2]
    //             },
    //             {
    //                 "section": 4,
    //                 "startTime": wyl_startTime4[3],
    //                 "endTime": wyl_endTime4[3]
    //             },
    //             {
    //                 "section": 5,
    //                 "startTime": wyl_startTime4[4],
    //                 "endTime": wyl_endTime4[4]
    //             },
    //             {
    //                 "section": 6,
    //                 "startTime": wyl_startTime4[5],
    //                 "endTime": wyl_endTime4[5]
    //             },
    //             {
    //                 "section": 7,
    //                 "startTime": wyl_startTime4[6],
    //                 "endTime": wyl_endTime4[6]
    //             },
    //             {
    //                 "section": 8,
    //                 "startTime": wyl_startTime4[7],
    //                 "endTime": wyl_endTime4[7]
    //             },
    //             {
    //                 "section": 9,
    //                 "startTime": wyl_startTime4[8],
    //                 "endTime": wyl_endTime4[8]
    //             },
    //             {
    //                 "section": 10,
    //                 "startTime": wyl_startTime4[9],
    //                 "endTime": wyl_endTime4[9]
    //             }
    //         ];
    //     } else if (wyl_id[1] == 8) {
    //         wyl_sectionTime = [{
    //                 "section": 1,
    //                 "startTime": wyl_startTime3[0],
    //                 "endTime": wyl_endTime3[0]
    //             },
    //             {
    //                 "section": 2,
    //                 "startTime": wyl_startTime3[1],
    //                 "endTime": wyl_endTime3[1]
    //             },
    //             {
    //                 "section": 3,
    //                 "startTime": wyl_startTime3[2],
    //                 "endTime": wyl_endTime3[2]
    //             },
    //             {
    //                 "section": 4,
    //                 "startTime": wyl_startTime3[3],
    //                 "endTime": wyl_endTime3[3]
    //             },
    //             {
    //                 "section": 5,
    //                 "startTime": wyl_startTime3[4],
    //                 "endTime": wyl_endTime3[4]
    //             },
    //             {
    //                 "section": 6,
    //                 "startTime": wyl_startTime3[5],
    //                 "endTime": wyl_endTime3[5]
    //             },
    //             {
    //                 "section": 7,
    //                 "startTime": wyl_startTime3[6],
    //                 "endTime": wyl_endTime3[6]
    //             },
    //             {
    //                 "section": 8,
    //                 "startTime": wyl_startTime3[7],
    //                 "endTime": wyl_endTime3[7]
    //             },
    //             {
    //                 "section": 9,
    //                 "startTime": wyl_startTime3[8],
    //                 "endTime": wyl_endTime3[8]
    //             },
    //             {
    //                 "section": 10,
    //                 "startTime": wyl_startTime3[9],
    //                 "endTime": wyl_endTime3[9]
    //             }
    //         ];
    //     } else if (wyl_id[1] == 9) {
    //         wyl_sectionTime = [{
    //                 "section": 1,
    //                 "startTime": wyl_startTime2[0],
    //                 "endTime": wyl_endTime2[0]
    //             },
    //             {
    //                 "section": 2,
    //                 "startTime": wyl_startTime2[1],
    //                 "endTime": wyl_endTime2[1]
    //             },
    //             {
    //                 "section": 3,
    //                 "startTime": wyl_startTime2[2],
    //                 "endTime": wyl_endTime2[2]
    //             },
    //             {
    //                 "section": 4,
    //                 "startTime": wyl_startTime2[3],
    //                 "endTime": wyl_endTime2[3]
    //             },
    //             {
    //                 "section": 5,
    //                 "startTime": wyl_startTime2[4],
    //                 "endTime": wyl_endTime2[4]
    //             },
    //             {
    //                 "section": 6,
    //                 "startTime": wyl_startTime2[5],
    //                 "endTime": wyl_endTime2[5]
    //             },
    //             {
    //                 "section": 7,
    //                 "startTime": wyl_startTime2[6],
    //                 "endTime": wyl_endTime2[6]
    //             },
    //             {
    //                 "section": 8,
    //                 "startTime": wyl_startTime2[7],
    //                 "endTime": wyl_endTime2[7]
    //             },
    //             {
    //                 "section": 9,
    //                 "startTime": wyl_startTime2[8],
    //                 "endTime": wyl_endTime2[8]
    //             },
    //             {
    //                 "section": 10,
    //                 "startTime": wyl_startTime2[9],
    //                 "endTime": wyl_endTime2[9]
    //             }
    //         ];
    //     }
    // } else if (wyl_id[0] == 2) {
    //     if (wyl_id[1] == 0) {
    //         wyl_sectionTime = [{
    //                 "section": 1,
    //                 "startTime": wyl_startTime1[0],
    //                 "endTime": wyl_endTime1[0]
    //             },
    //             {
    //                 "section": 2,
    //                 "startTime": wyl_startTime1[1],
    //                 "endTime": wyl_endTime1[1]
    //             },
    //             {
    //                 "section": 3,
    //                 "startTime": wyl_startTime1[2],
    //                 "endTime": wyl_endTime1[2]
    //             },
    //             {
    //                 "section": 4,
    //                 "startTime": wyl_startTime1[3],
    //                 "endTime": wyl_endTime1[3]
    //             },
    //             {
    //                 "section": 5,
    //                 "startTime": wyl_startTime1[4],
    //                 "endTime": wyl_endTime1[4]
    //             },
    //             {
    //                 "section": 6,
    //                 "startTime": wyl_startTime1[5],
    //                 "endTime": wyl_endTime1[5]
    //             },
    //             {
    //                 "section": 7,
    //                 "startTime": wyl_startTime1[6],
    //                 "endTime": wyl_endTime1[6]
    //             },
    //             {
    //                 "section": 8,
    //                 "startTime": wyl_startTime1[7],
    //                 "endTime": wyl_endTime1[7]
    //             },
    //             {
    //                 "section": 9,
    //                 "startTime": wyl_startTime1[8],
    //                 "endTime": wyl_endTime1[8]
    //             },
    //             {
    //                 "section": 10,
    //                 "startTime": wyl_startTime1[9],
    //                 "endTime": wyl_endTime1[9]
    //             }
    //         ];
    //     }
    // }
    console.log(wyl_sectionTime)
    return {
        courseInfos: result,
        sectionTimes: wyl_sectionTime
    }
}
```

以上，如果哪里有问题请联系我~

迭代记录：
- v1.0：上线
- v1.1：修复了周六形式与政策的bug
- v2.0：修复了单周的bug，填上注释，修改代码规范，方便重构。
- v2.1：修复了并列单周如果出现两位数字的时候只会读取第一位，原因是直接把切完的数组的第一位放进去了，也不知道自己当时怎么想的- -，又切了一次，把周切掉放进去就fix啦~感谢浙农林的zeyu同学报的bug~
- v3.0：换了一版时间，原来的东西都注释掉了，还是担心学校又改时间，到时候不方便改，也为了给大家留个参考。