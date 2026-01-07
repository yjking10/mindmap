<template>

  <!-- <button @click="exportToJpeg">Export to JPEG</button> -->
  <!-- <div className="control">
      <div class="button-svg" @click="zoomIn">
        zoomIn
      </div>

      <div class="button-svg" @click="zoomOut">
        zoomOut
      </div>

      <div class="button-svg" @click="onFit">
        onFit
      </div>
    </div> -->

  <div ref="markmapContainer" class="markmap-container">
    <svg id="markmap" ref="svgRef"></svg>
  </div>

</template>

<script setup>

import { ref, onMounted, onUpdated, onUnmounted, onBeforeMount, nextTick } from 'vue';
import { Markmap } from 'markmap-view';
import { transformer } from '../lib/markmap';

import html2canvas from 'html2canvas';

const svgRef = ref();
const markmapContainer = ref()
const mdContent = ref('');
let mm;
// const markdownContent = `# Markmap
// # React 项目
// ## 安装
// - npm install
// - yarn add
// ## 组件
// - Button
// - Input
// - Modal
// ## API 调用
// - fetch
// - axios
// - dddd
// `;
onMounted(() => {
  //   initMarkmap(markdownContent);
  // 初始化 Markmap
  mm = Markmap.create(svgRef.value, { initialExpandLevel: 3, fitRatio: 1, spacingVertical: 16 });

  // setMindMapContent(markdownContent)
});




// const initMarkmap = (content) => {
//   mdContent.value = content;
//   if (!markmapContainer.value) return


//   // 创建 SVG 容器
//   // svgRef.value = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
//   // svgRef.value.classList.add('mm-svg');
//   // markmapContainer.value.appendChild(svgRef.value);


//   update();
// }


const update = async () => {
  const { root } = transformer.transform(mdContent.value);
  const filteredRoot = filterImageNodes(root);
  await mm.setData(filteredRoot);
  mm.fit();
  // setTimeout(() => {

  // }, 1000);
}

const filterImageNodes = (node) => {
  if (typeof node.content === "string") {
    node.content = node.content.replace(/<img[^>]*>/g, "").trim();
  }
  if (Array.isArray(node.children)) {
    node.children = node.children
      .map(filterImageNodes)
      .filter(
        (child) =>
          child != null &&
          ((typeof child.content === "string" &&
            child.content.length > 0) ||
            (Array.isArray(child.children) && child.children.length > 0))
      );
  }
  return node;
}





// 暴露一个全局函数供 Flutter 调用
const setMindMapContent = (message) => {

  mdContent.value = message;
  update();

  return true;
};
const zoomIn = async () => {
  await mm.rescale(1.2);

  return true;
};

const zoomOut = async () => {
  await mm.rescale(0.8);

  return true;
};

const onFit = async () => {

  await mm.fit();
  return true;
};

const exportToJpeg = () => {



  exportToImage();

};



const exportToImage = async () => {
  try {

    await onFit();
    await nextTick()

    let scale = window.devicePixelRatio

    const options = {
      scale: scale,                  // 缩放倍数（提高分辨率）
      useCORS: true,            // 允许加载跨域资源
      allowTaint: true,         // 允许污染画布（需配合useCORS）
      backgroundColor: '#fff',   // 强制白色背景
      logging: false,           // 关闭调试日志

    };
    // 使用 html2canvas 截图
    const canvas = await html2canvas(markmapContainer.value, options);

    // console.log(canvas.toDataURL('image/jpeg'));


    sendMessageToNative({ image: canvas.toDataURL('image/jpeg').split(',')[1] })

  } catch (err) {
    console.error('导出失败:', err);
  }
}



onBeforeMount(() => {
  // window.initMarkmap = initMarkmap;
  window.setMindMapContent = setMindMapContent;
  window.zoomIn = zoomIn;
  window.zoomOut = zoomOut;
  window.onFit = onFit;
  window.exportToJpeg = exportToJpeg;
});



const sendMessageToNative = (message) => {

  // 针对 iOS
  if (window.webkit && window.webkit.messageHandlers && window.webkit.messageHandlers.waveNoteBridge) {
    window.webkit.messageHandlers.waveNoteBridge.postMessage(JSON.stringify(message));
  }
  // 针对 Android
  else if (window.waveNoteBridge) {
    window.waveNoteBridge.postMessage(JSON.stringify(message));
  } else {
    console.error("Native handler not available");
  }
}

onUpdated(update);


onUnmounted(() => {
  if (mm) {
    mm.destroy();
  }

})

</script>

<style lang="scss">
.markmap-container {
  width: 100vw;
  height: 100vh;
  background-color: #fff;
  position: relative;
  display: flex;
}


#markmap {
  width: 100%;
  height: 100%;
  background-color: #fff;

  .markmap {
    background-color: #fff;
    color: #fff;
  }

  .markmap-node {
    cursor: pointer;
    fill: #20202C;
    color: #20202C;
  }
}


// background-color: var(--color-background);</style>
