<script setup>

const streams = usePeersStreams();

onMounted(() => {
    setTimeout(()=>{ 
        MediaControl.show();
        MediaControl.getDevices();},100)


});
</script>
<template>
  <i-row center style="background-image:url('./background.jpg');background-size:100%;height:85vh;padding:10px;margin:0px" >
    <i-column xs="12" md="8" lg="6" color="dark" >
    <div style="margin: 3px"
      v-for="(item, index) in streams"
      :key="'stream_' + index">
        <MediaVideo :stream="item.stream" />
    </div>
    </i-column>
        <i-column xs="12" md="2" lg="2" >
            <div  >
      <MediaVideoSelf />
      </div>
    </i-column>
  </i-row>
</template>