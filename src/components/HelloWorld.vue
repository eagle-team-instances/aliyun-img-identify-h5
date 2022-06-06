<template>
  <div class="hello">
    <van-divider
      ><h1><van-icon name="cluster-o" />垃圾分类</h1></van-divider
    >
    <van-card v-for="(cell, index) in cells.image" :key="index">
      <template #title>
        <div v-show="cell.info" class="gb-msg">
          <div style="width: 100%; display: flex; flex-wrap: wrap">
            <div class="gb-row">
              垃圾类别置信度: {{ cell.info?.CategoryScore }}
            </div>
            <div class="gb-line">
              名称: {{ cell.info?.Rubbish || "什么垃圾?😅" }}
            </div>
            <div class="gb-row">
              物品名称置信度: {{ cell.info?.RubbishScore }}
            </div>
            <div class="gb-line">
              垃圾类别:
              <van-tag type="primary">{{
                cell.info?.Category || "你看着扔吧"
              }}</van-tag>
            </div>
          </div>
        </div>
      </template>
      <!-- <template #desc> {{ cell.info?.Rubbish }} </template> -->
      <template #thumb>
        <van-uploader
          v-model="cell.fileList"
          :deletable="false"
          :after-read="afterRead"
          @click-preview="showResult"
          :max-count="1"
        >
          <template #preview-cover="{ file }">
            <div class="preview-cover van-ellipsis">{{ file.name }}</div>
          </template>
        </van-uploader>
      </template>
    </van-card>
  </div>
</template>

<script>
import { reactive } from "vue";
import axios from "axios";

const HOST_NAME = "http://192.168.76.23:5000/file/upload";

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  setup() {
    const cells = reactive({ image: [{}] });
    async function afterRead(file) {
      const rawData = new FormData();
      rawData.append("file", file.file);
      file.status = "uploading";
      const { data: aiIdtRes } = await axios({
        method: "POST",
        url: HOST_NAME,
        data: rawData,
        headers: {
          "Content-Type": "multipart/form-data",
        },
      });
      file.status = "done";
      const aimData = aiIdtRes.data[0].Data.Elements[0];
      cells.image[cells.image.length - 1].info = {
        Category: aimData.Category,
        CategoryScore: parseFloat(aimData.CategoryScore).toFixed(2),
        Rubbish: aimData.Rubbish,
        RubbishScore: parseFloat(aimData.RubbishScore).toFixed(2),
      };
      // aiIdtRes.data[0].Data.Elements[0];
      // cells[cells.length - 1].value.push();
      // console.log(aiIdtRes.data[0].Data.Elements[0]);
      cells.image.push({});
      // cells.image.push(aiIdtRes.data[0].Data.Elements[0]);
      // cells.image.push(aiIdtRes.data[0].Data.Elements[0]);
    }

    function showResult() {}

    return {
      cells,
      afterRead,
      showResult,
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.preview-cover {
  position: absolute;
  bottom: 0;
  box-sizing: border-box;
  width: 100%;
  padding: 4px;
  color: #fff;
  font-size: 12px;
  text-align: center;
  background: rgba(0, 0, 0, 0.3);
}
.gb-msg {
  display: flex;
  // flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  height: 88px;
}
.gb-row {
  width: 50%;
  margin: 3px 0px;
}

.gb-line {
  margin: 3px 0px;
  display: flex;
  align-items: center;
}
</style>
