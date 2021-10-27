<template>
  <div id="app">
      <h1>
        DVS Demo Application
      </h1>
      <div id="videoCapturingEl"></div>
  </div>
</template>

<script>
import IDVC from '@idscan/idvc';

export default {
  name: 'App',
  mounted: function() {
        new IDVC({
            el: 'videoCapturingEl',
            licenseKey: 'LICENSE KEY REPLACE ME',
            isShowManualSwitchButton: true,
            showSubmitBtn: true,
            isShowVersion: true,
            tapOnVideo: false,
            tapBackSide: false,
            minPDFframes: 1,
            parseMRZ: false,
            tapFace: false,
            enableLimitation: true,
            autoContinue: true,
            resizeUploadedImage: 1500,
            showForceCapturingBtn: false,
            fixFrontOrientAfterUpload: false,
            enableFlash: false,
            capturingMode: '4',
            steps: [
                { type: 'front', name: 'Front Scan' },
                { type: 'face', name: 'Selfie' },
            ],
            useCDN: true,
            networkUrl: '/assets/networks',
            showPreviewForOneStep: true,
            priority: 'auto',
            realFaceMode: 'all',
            types: ['ID'],
            strictAllowedTypes: false,
            enableGeolocation: false,
            displayParsedData: false,
            onChange(data) {
                console.log(data);
            },
            onCameraError(data) {
                console.log(data);
            },
            onReset(data) {
                console.log(data);
            },
            onRetakeHook(data) {
                console.log(data);
            },
            submit(data) {
                let backStep = data.steps.find((item) => item.type === 'back');
                let trackString =
                backStep && backStep.trackString ? backStep.trackString : '';

                let request = {
                frontImageBase64: data.steps
                    .find((item) => item.type === 'front')
                    .img.split(/:image\/(jpeg|png);base64,/)[2],
                backOrSecondImageBase64: backStep.img.split(
                    /:image\/(jpeg|png);base64,/
                )[2],
                faceImageBase64: data.steps
                    .find((item) => item.type === 'face')
                    .img.split(/:image\/(jpeg|png);base64,/)[2],
                documentType: data.documentType,
                trackString: trackString,
                userAgent: window.navigator.userAgent,
                captureMethod: data.captureMethod,
                verifyFace: true,
                };

                fetch('https://dvs2.idware.net/api/Request', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json;charset=utf-8',
                    Authorization: 'BEARER TOKEN REPLACE ME',
                },
                body: JSON.stringify(request),
                })
                .then((response) => response.json())
                .then((response) => {
                    fetch('BACKEND SERVER URL REPLACE ME', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json;charset=utf-8',
                    },
                    body: JSON.stringify({
                        requestId: response.requestId,
                    }),
                    })
                    .then((response) => response.json())
                    .then((data) => {
                        console.log(data);
                    });
                })
                .catch((err) => {
                    console.log(err);
                });
            },
        });
  }
}

</script>

<style>
@import '../node_modules/@idscan/idvc/dist/css/idvc.css';

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
