<template>
  <div id="app">
      <h1>
        DVS Demo Application
      </h1>
      <div id="videoCapturingEl"></div>
  </div>
</template>

<script>
import IDVC from '@idscan/idvc2';

export default {
  name: 'App',
  mounted: function() {
        let idvc = new IDVC({
            el: "videoCapturingEl",
            licenseKey: "LICENSE_KEY",
            networkUrl: "networks",
            chunkPublicPath: "networks",
            resizeUploadedImage: 1200,
            fixFrontOrientAfterUpload: false,
            autoContinue: true,
            isShowDocumentTypeSelect: false,
            useCDN: false,
            isShowGuidelinesButton: false,
            isSubmitMetaData: false,
            useHeic: false,
            showSubmitBtn: false,
            hideDocumentTitle: false,
            language: "en",
            realFaceMode: "auto",
            modalPosition: 'top',
            processingImageFormat: 'jpeg',
            documentTypes: [
                {
                type: "ID",
                steps: [
                        {
                        type: "front",
                        name: "Document Front",
                        },
                        {
                        type: "pdf",
                        name: "Document Back",
                        },
                        {
                        type: "face",
                        name: "Face",
                        },
                    ],
                },
                {
                type: "Passport",
                steps: [
                    {
                    type: "mrz",
                    name: "Passport Front",
                    },
                    {
                    type: "face",
                    name: "Face",
                    },
                    ],
                }
            ],
            onChange(data) {
                console.log("on change", data);
            },
            onCameraError(data) {
                console.log("camera error", data);
            },
            onReset(data) {
                console.log("on reset", data);
            },
            onRetakeHook(data) {
                console.log("retake hook", data);
            },
            clickGuidlines() {
                console.log("click Guidelines");
            },
            submit(data) {
                idvc.showSpinner(true);
                let frontStep, pdfStep, faceStep, mrzStep;
                let frontImage, backImage, faceImage;
                let rawTrackString;


                switch (data.documentType) {
                // Drivers License and Identification Card
                case 1:
                    frontStep = data.steps.find((item) => item.type === "front");
                    pdfStep = data.steps.find((item) => item.type === "pdf");
                    faceStep = data.steps.find((item) => item.type === "face");

                    frontImage = frontStep.img.split(/:image\/(jpeg|png|webp);base64,/)[2];
                    backImage = pdfStep.img.split(/:image\/(jpeg|png|webp);base64,/)[2];
                    faceImage = faceStep.img.split(/:image\/(jpeg|png|webp);base64,/)[2];

                    rawTrackString = pdfStep && pdfStep.trackString ? pdfStep.trackString : "";

                    break;
                // US and International Passports
                case 2:
                    mrzStep = data.steps.find((item) => item.type === "mrz");
                    faceStep = data.steps.find((item) => item.type === "face");

                    frontImage = mrzStep.img.split(/:image\/(jpeg|png|webp);base64,/)[2];
                    faceImage = faceStep.img.split(/:image\/(jpeg|png|webp);base64,/)[2];

                    rawTrackString = mrzStep && mrzStep.mrzText ? mrzStep.mrzText : "";

                    break;
                default:
                }

                const trackStringArray = rawTrackString.split(".");
                let trackString = trackStringArray[0];
                let barcodeParams = trackStringArray[1];

                let request = {
                    frontImageBase64: frontImage,
                    backOrSecondImageBase64: backImage,
                    faceImageBase64: faceImage,
                    documentType: data.documentType,
                    trackString:{
                        data:  trackString,
                        barcodeParams: barcodeParams
                    },
                    overriddenSettings: {
                        isOCREnabled: true,
                        isBackOrSecondImageProcessingEnabled: true,
                        isFaceMatchEnabled: true
                    }
                };

                fetch("https://dvs2.idware.net/api/v4/verify", {
                method: "POST",
                headers: {
                    Authorization: "Bearer SECRET_KEY",
                    "Content-Type": "application/json;charset=utf-8",
                },
                body: JSON.stringify(request),
                })
                .then((response) => response.json())
                .then((data) => {
                    idvc.showSpinner(false);
                    console.log(data);
                })
                .catch((err) => {
                    idvc.showSpinner(false);
                    console.log(err);
                });
            },
        });
  }
}

</script>

<style>
@import '../node_modules/@idscan/idvc2/dist/css/idvc.css';

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
