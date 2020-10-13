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

let data =  {
    publicKey: '***REMOVED***',
    backendServerUrl: '***REMOVED***',
    licenseKey: '***REMOVED***'
  };

export default {
  name: 'App',
  data: data,
  mounted: function() {
      new IDVC({
        el: 'videoCapturingEl',
        networkUrl: '/assets/networks',
        tapBackSide : true,
        licenseKey: this.licenseKey,
        steps: [
            {type: 'front', name: 'Front Scan'},
            {type: 'back', name: 'Back Scan'},
            {type: 'face', name: 'Selfie'}
        ],
        submit (data) {
            let backStep = data.steps.find(item => item.type === 'back')
            let trackString = (backStep && backStep.trackString) ? backStep.trackString : ''

            let request = {
                frontImageBase64: data.steps.find (item => item.type === 'front').img.split (/:image\/(jpeg|png);base64,/)[2],
                backOrSecondImageBase64: backStep.img.split (/:image\/(jpeg|png);base64,/)[2],
                faceImageBase64: data.steps.find (item => item.type === 'face').img.split (/:image\/(jpeg|png);base64,/)[2],
                documentType: data.documentType,
                trackString: trackString
            }
            
            fetch ('https://dvs2.idware.net/api/Request', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json;charset=utf-8',
                    'Authorization': `Bearer ${this.publicKey}`
                },
                body: JSON.stringify (request)
            }).then (response => response.json ())
                .then (response => {
                    fetch (this.backendServerUrl + '/api/ValidationRequests/complete/', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json;charset=utf-8'
                        },
                        body: JSON.stringify ({
                            requestId: response.requestId,
                            documentType: response.documentType
                        })
                    }).then (response => response.json ())
                        .then (data => {
                            
                            alert((data.payload.isDocumentSuccess) ? 'Document valid' : 'Document invalid')
                        })
                }).catch(() => {
                
            })
        }
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
