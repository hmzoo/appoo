<script setup>
const {
  $initPeer,
  $cleanPeer,
  $interfacePeer,
  $getConn,
  $getMedia,
  $connectPeer,
  $heartBeat,
  $callPeer,
  $closeMedia,
  $setssid,
} = useNuxtApp();
useHead({
  titleTemplate: "OOCAM",
  viewport: "width=device-width, initial-scale=1, maximum-scale=1",
  charset: "utf-8",
  meta: [{ name: "description", content: "My amazing site." }],
});
const route = useRoute();
const showsidebar = ref(false);

const switchSideBar = () => {
  showsidebar.value = !showsidebar.value;
};

const getSlug = () => {
  return route.path.replace(/\//g, "");
};
const checkSlug = () => {
  console.log("chekslug", getSlug() == "");
  return getSlug() == "";
};

const checkF = () => {
  return ufuk.value.length > 0;
};

const ubc = useBrowserConf();
const ufuk = useFollowsUKey();
const ukey = useUKey();
const uss = useSelfStream();

const callPeer = (id) => {
  if (uss.value != null) {
    logit("PEER", "CALLPEER STREAM " + id + "USS:" + uss.value);
    $callPeer(id, uss.value);
  }
};

const connectPeer = (id) => {
  logit("PEER", "CONNECTPEER " + id);
  $connectPeer(id);
};

const closeMedia = (id) => {
  logit("PEER", "CLOSEMEDIA " + id);
  delPeerStream(id);
  $closeMedia(id);
};

const startApp = () => {
  logit("APP", "STARTING");
  getApiUKey();

  if (typeof $interfacePeer !== "undefined") {
    $interfacePeer.onPeerOpen = (id) => {
      logit("PEER", "ONPEEROPEN " + id);
      peerStatus("PEER READY " + id);
      setInfoUKey(id);
    };

    $interfacePeer.onPeerConn = (conn) => {
      logit("PEER", "ONPEERCONN " + conn.peer);
      updateApp();
      setConnFollow(conn, "CONNECTION");
    };

    $interfacePeer.onConnData = (conn, data) => {
      setConnFollow(conn, "");
      console.log("DATA", data);
      if (data.msg != undefined) {
        logit("PEER", "ONCONNDATA " + conn.peer + " " + data.msg);
        addPeerMessage(conn.peer, data.msg);
        setConnFollow(conn, "MSG");
      }
      if (data.sr != undefined && data.sr == "callme") {
        callPeer(conn.peer);
      }
      if (data.ssid != undefined) {
        if (!checkssid(conn.peer, data.ssid)) {
           closeMedia(conn.peer);
        }
      }
    };
    $interfacePeer.onConnOpen = (conn) => {
      logit("PEER", "ONCONNOPEN " + conn.peer);
      setConnFollow(conn, "OPEN");
    };
    $interfacePeer.onConnClose = (conn) => {
      logit("PEER", "ONCONNCLOSE " + conn.peer);
      setConnectedFollowUkey(getFollowFromInfo(conn.peer).ukey, false);
      setConnFollow(conn, "CLOSE");
      closeMedia(conn.peer);
    };
    $interfacePeer.onConnError = (conn, err) => {
      logit("PEER", "ONCONNERROR " + conn.peer + " " + err.toString());
      Console.log(
        "Peer connection err",
        id,
        err.toString(),
        getFollowFromInfo(id)
      );
      setConnFollow(conn, "ERROR");
      closeMedia(conn.peer);
    };

    $interfacePeer.onPeerCall = (media) => {
      logit("PEER", "ONPEERCALL " + media.peer);
      setMediaFollow(media, "CALL");
      if (selfStreamIsOn()) {
        logit("PEER", "ANSWER WITH STREAM " + media.peer);
        media.answer(getSelfStream());
      } else {
        logit("PEER", "ANSWER NO STREAM " + media.peer);
        media.answer();
      }
      //media.answer(uss.value)
    };

    $interfacePeer.onMediaStream = (media, stream) => {
      logit("PEER", "ONMEDIASTREAM " + media.peer);
      console.log("ONSTREAM", stream);
      setMediaFollow(media, "STREAM");
      setPeerStream(media.peer, stream);
    };
    $interfacePeer.onMediaClose = (media) => {
      logit("PEER", "ONMEDIACLOSE " + media.peer);
      setMediaFollow(media, "CLOSE");
      closeMedia(media.peer);
      //setConnectedFollowUkey(getFollowFromInfo(id).ukey, false);
    };
    $interfacePeer.onMediaError = (media, err) => {
      logit("PEER", "ONMEDIAERROR " + media.peer + " " + err.toString());
      setMediaFollow(media, "ERROR");
      closeMedia(media.peer);
    };
    $interfacePeer.onStreamInactive = (id) => {
      logit("PEER", "ONSTREAMINACTIVE " + id);
    };
    $interfacePeer.onStreamActive = (id) => {
      logit("PEER", "ONSTREAMACTIVE " + id);
    };
    MediaControl.onStreamOn = () => {
      $setssid(uss.value.id);
      ufuk.value.forEach((f) => {
        $callPeer(f.info, uss.value);
      });
    };
    MediaControl.onStreamOff = () => {$setssid("")};
    usePeersMessages().value = [];
    usePeersStreams().value = [];

    setTimeout($initPeer, 300);
    setTimeout(updateApp, 1000);
  }
  // setTimeout(checkFollowers, 1000);
};

const updateApp = () => {
  logit("APP", "UPDATING");
  getAllDataUkey();
  setTimeout(checkFollowers, 1000);
  setTimeout(checkConns, 2000);
};

const resetApp = () => {
  showsidebar.value = false;
  logit("APP", "RESETING");
  $cleanPeer();
  resetUKey();
  startApp();
};
const checkConns = () => {
  logit("APP", "CHECKCONNS");
  useFollowsUKey().value.forEach((f) => {
    if ($getConn(f.info) == null && f.info != "DOWN") {
      connectPeer(f.info);
    }
  });
};

const addFollow = (d) => {
  console.log("rrr", d);
  addFollowUkey(d);
  setTimeout(updateApp, 1000);
};

onMounted(() => {
  window.cookieconsent.initialise({
    palette: {
      popup: {
        background: "#edeff5",
        text: "#838391",
      },
      button: {
        background: "#4b81e8",
      },
    },
  });
  startApp();
  setInterval(updateApp, 20000);
  setInterval($heartBeat, 3000);
});
</script>

<template>
  <i-layout>
    <i-layout-header>
      <i-navbar
        size="sm"
        :collapse="false"
        class="_background:primary"
        style="padding: 0; margin: 0"
      >
        <i-navbar-brand>
          <span class="title">OOCAM</span> ID: {{ ukey.ukey }}
        </i-navbar-brand>

        <i-nav size="sm" v-if="checkF()">
          <i-nav-item>
            <nuxt-icon
              name="EnabCam"
              class="iconbtn"
              v-if="ubc.camera_status"
              @click="MediaControl.switchCam()"
            />
            <nuxt-icon
              name="DisabCam"
              class="iconbtn"
              v-else
              @click="MediaControl.switchCam()"
            />
          </i-nav-item>
          <i-nav-item>
            <nuxt-icon
              name="EnabMic"
              class="iconbtn"
              v-if="ubc.micro_status"
              @click="MediaControl.switchMic()"
            />
            <nuxt-icon
              name="DisabMic"
              class="iconbtn"
              v-else
              @click="MediaControl.switchMic()"
            />
          </i-nav-item>
          <i-nav-item>
            <nuxt-icon name="settings" class="iconbtn" @click="switchSideBar" />
          </i-nav-item>
        </i-nav>
      </i-navbar>
    </i-layout-header>
    <i-layout vertical class="_padding-top:1/2">
      <i-sidebar
        v-model="showsidebar"
        :collapse="true"
        collapse-position="absolute"
        placement="right"
        :collapseOnItemClick="false"
      >
        <Settings @onResetNumber="resetApp" />
      </i-sidebar>
      <i-layout-content>
        <i-container fluid style="height: 90vh; padding: 0; margin: 0">
          <Welcome v-if="!checkF()" @onCallNumber="addFollow" />
          <Main v-else />
        </i-container>
      </i-layout-content>
    </i-layout>
  </i-layout>
</template>

<style lang="scss">
@import url("https://cdn.jsdelivr.net/npm/cookieconsent@3/build/cookieconsent.min.css");
@import url("https://fonts.googleapis.com/css2?family=Roboto:wght@900&family=Rubik+Moonrocks&family=Secular+One&display=swap");
@import "@inkline/inkline/css/variables";
@import "@inkline/inkline/css/mixins";

@media (prefers-color-scheme: dark) {
}

@include i-navbar() {
  ----item--color: white;
  @include variant("light") {
    ----padding: 0;
    ----margin: 0;
    ----background: primary;
  }
}
.title {
  font-family: "Secular One", sans-serif;
  color: white;
  font-size: 2em;
  font-weight: 900;
}

.iconbtn {
  font-size: 2em;
  margin: 0;
  padding: 0;
  cursor: pointer;
  color: white;
}
.videocard.card {
  --padding-top: 0.4rem;
  --padding-bottom: 0;
  --padding-right: 0;
  --padding-left: 0;
  ----background: var(--color--dark);
}

.videocard.-light {
  --background: primary;
}

.main.-light {
  --background: primary;
}
</style>

