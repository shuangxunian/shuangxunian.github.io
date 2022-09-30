---
title: 一篇关于支付的文章
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- vue
- js
---

## 前言
用 vue + element 实现了一个支付，后来改 UI 了这个就不要了，删除关键发出来，发完改版去- -

## code
```html
    <!-- 充值 -->
    <el-dialog title="充值" :visible.sync="payDialogVisible" width=500px @close="payDialogClosed" :close-on-click-modal = "false">

      <el-form :model="payForm" :rules="payRules" ref="payFormRef" v-loading="fullscreenLoading" label-position="left">
        <p class="myPayText">欢迎使用<span style="color:rgb(21,119,255)">支付宝</span></p>
        <el-form-item style="margin-top:20px" label="充值次数：" label-width="140px" prop="stuNewPassword" >
          <el-input-number v-model="payForm.num" label="描述文字" :min="1"></el-input-number>
          <span style="padding-left:60px">共计{{this.payForm.num * 5}}元</span>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="payDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="makeSurePay">确 定</el-button>
      </div>

    </el-dialog>

    <!-- 扫码 -->
    <el-dialog title="充值" :visible.sync="codeDialogVisible" width="30%" @close="codeDialogClosed" :close-on-click-modal = "false">

      <el-form :rules="codeRules" ref="codeFormRef">
        <p style="font-size:24px">订单编号：<b>{{orderNumber}}</b></p>
        <p style="font-size:30px;">请使用<span style="color:rgb(21,119,255)">支付宝</span>扫码支付</p>
        <img
          style="width:150px;display:block;margin:0 auto;margin-top:20px"
          alt="暂无数据"
          :src="'data:image/png;base64,'+myBase64"
        >
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="codeDialogVisible = false">取 消</el-button>
        <!-- <el-button type="primary" @click="makeSureCode">支付完成</el-button> -->
      </div>

    </el-dialog>

<script>
    makeSurePay () {
      this.$refs.payFormRef.validate(async (valid) => {
        if (!valid) return
        this.fullscreenLoading = true
        setTimeout(() => {
          this.orderNumber = Math.ceil(Math.random() * 100000000000)
          this.myBase64 = '这里是向后端要的base64'
          this.fullscreenLoading = false
          this.codeDialogVisible = true
          this.payDialogVisible = false
        }, 2000)
      })
    },
</script>
```

说明：这里只是思路，不是针对萌新的文章，遮罩层那些基础的东西就自己写一下就好。

