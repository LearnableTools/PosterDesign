<!--
 * @Author: ShawnPhang
 * @Date: 2021-08-01 11:12:17
 * @Description: 前端出图 - 用于封面
 * @LastEditors: ShawnPhang <site: book.palxp.com>
 * @LastEditTime: 2023-07-26 10:22:17
-->
<template>
  <div id="cover-wrap">
    <img id="cover" />
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, watch, getCurrentInstance, ComponentInternalInstance } from 'vue'
import { mapGetters, mapActions } from 'vuex'
import html2canvas from 'html2canvas'
import api from '@/api'
import Qiniu from '@/common/methods/QiNiu'

export default defineComponent({
  props: ['modelValue'],
  emits: ['update:modelValue'],
  setup(props, context) {
    let gridSizeIndex: number = 0

    const { proxy }: any = getCurrentInstance() as ComponentInternalInstance

    async function createCover(cb: any) {
      const nowGrideSizeIndex = gridSizeIndex
      const nowZoom = proxy?.dZoom
      // 取消选中元素
      proxy?.selectWidget({
        uuid: '-1',
      })
      gridSizeIndex = 0
      proxy?.updateZoom(100)
      const opts = {
        useCORS: true, // 跨域图片
        scale: 0.2,
      }
      setTimeout(async () => {
        html2canvas(document.getElementById('page-design-canvas'), opts).then((canvas: any) => {
          canvas.toBlob(
            async (blobObj: Blob) => {
              const result: any = await Qiniu.upload(blobObj, { bucket: 'xp-design', prePath: 'cover/user' })
              cb(result)
            },
            'image/jpeg',
            0.1,
          )
          gridSizeIndex = nowGrideSizeIndex
          proxy?.updateZoom(nowZoom)
        })
      }, 10)
    }

    return {
      createCover,
    }
  },
  computed: {
    ...mapGetters(['dZoom']),
  },
  methods: {
    ...mapActions(['selectWidget', 'updateZoom']),
  },
})
</script>

<style lang="less" scoped>
#cover-wrap {
  position: absolute;
  left: -9999px;
}
</style>
