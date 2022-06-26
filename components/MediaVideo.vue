<script setup>
const props = defineProps({
  stream: { Type: Object },
  muted:{Type : String }
});


const testStream=(stream)=>{
  console.log("streamtest",stream)
}

</script>
<template>
<i-card class="videocard " >

        <video @click="testStream(selfStream)"
          ref="streamVideo"
          style="object-fit: contain;width: 100%    !important; height: 100%   !important;"
          autoplay="autoplay"
           :muted="muted"
          :srcObject="stream"
        ></video>
        
        </i-card>
</template>