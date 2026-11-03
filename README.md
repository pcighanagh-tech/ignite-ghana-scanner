<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#172b4d">
<title>IGNITE GHANA 2026 — QR Scanner</title>
<script src="https://unpkg.com/html5-qrcode"></script>
<style>
*{box-sizing:border-box}body{margin:0;padding:18px;background:#f3f5f8;color:#172033;font-family:Arial,sans-serif}
main{max-width:620px;margin:0 auto;background:#fff;border:1px solid #e1e6ee;border-radius:18px;padding:22px;box-shadow:0 5px 22px #17203312}
h1{font-size:23px;margin:0 0 7px;color:#172b4d}p{color:#647084;line-height:1.5}
label{display:block;font-weight:700;margin:18px 0 7px}select,input,button{width:100%;font:inherit;border-radius:10px;padding:14px}
select,input{border:1px solid #cbd3df;background:#fff}button{border:0;background:#1c2e52;color:#fff;font-weight:700;margin-top:14px;cursor:pointer}
button.secondary{background:#e9edf3;color:#172033}button:disabled{opacity:.5;cursor:not-allowed}
#reader{width:100%;margin:18px auto 8px;overflow:hidden}#status{padding:14px;border-radius:10px;background:#f1f5f9;line-height:1.5;margin-top:14px;overflow-wrap:anywhere}
#status.good{background:#eaf7ef;color:#12643b}#status.bad{background:#fff0ef;color:#a12820}.small{font-size:13px}
</style>
</head>
<body>
<main>
<h1>IGNITE GHANA 2026</h1>
<p>Secure QR attendance scanner. This scanner is connected to the signed-in staff dashboard on this page.</p>
<label for="day">Conference day</label>
<select id="day"><option>DAY 1</option><option>DAY 2</option><option>DAY 3</option></select>
<label for="device">Scanner/device label</label>
<input id="device" value="Entrance scanner" maxlength="80">
<div id="reader"></div>
<button id="start" type="button">Start camera scanner</button>
<button id="stop" class="secondary" type="button" disabled>Stop scanner</button>
<div id="status" role="status">Ready. Start the camera, then point it at an attendee's QR pass.</div>
<p class="small">When a QR code is recognised, its token is sent to the staff dashboard for attendance validation.</p>
</main>
<script>
(() => {
const statusBox=document.getElementById('status'),startButton=document.getElementById('start'),stopButton=document.getElementById('stop');
let qrScanner=null,busy=false;
function status(message,kind){statusBox.textContent=message;statusBox.className=kind||'';}
async function stopScanner(){if(!qrScanner)return;const current=qrScanner;qrScanner=null;try{if(current.isScanning)await current.stop();await current.clear();}catch(e){}startButton.disabled=false;stopButton.disabled=true;}
async function startScanner(){
 if(qrScanner)return;
 if(!window.isSecureContext||!navigator.mediaDevices||!navigator.mediaDevices.getUserMedia){status('Camera access requires a secure HTTPS page in a supported browser.','bad');return;}
 if(!window.Html5Qrcode){status('The QR scanning library did not load. Check your internet connection and reload this page.','bad');return;}
 startButton.disabled=true;status('Requesting camera access. Please allow the browser permission when prompted.');qrScanner=new Html5Qrcode('reader');
 const onScanSuccess=async(decodedText)=>{
  if(busy)return;busy=true;await stopScanner();
  const message={type:'IGNITE_QR_SCAN',token:String(decodedText).trim(),day:document.getElementById('day').value,deviceLabel:document.getElementById('device').value.trim()||'Entrance scanner'};
  if(!message.token){status('The QR code was empty. Start the scanner and try again.','bad');busy=false;return;}
  if(window.parent === window){status('This scanner must be displayed inside the signed-in staff dashboard. Return to the dashboard and tap Open live camera scanner.','bad');busy=false;return;}
  // Send the scan to the parent dashboard iframe host. The dashboard validates origin and frame source.
  window.parent.postMessage(message,'*');
  status('QR recognised and sent to the staff dashboard. Check the dashboard tab for the attendance result.','good');busy=false;
 };
 try{
  await qrScanner.start({facingMode:'environment'},{fps:10,qrbox:{width:250,height:250},aspectRatio:1},onScanSuccess);
  status('Camera is on. Point it steadily at the attendee’s QR code.');stopButton.disabled=false;
 }catch(error){
  const detail=error&&(error.name||error.message)?(error.name||error.message):String(error);
  await stopScanner();status('Could not start the camera: '+detail+'. Check browser camera permission and make sure no other app is using the camera.','bad');startButton.disabled=false;
 }
}
startButton.addEventListener('click',startScanner);
stopButton.addEventListener('click',async()=>{await stopScanner();status('Scanner stopped. You can start it again whenever you are ready.');});
})();
</script>
</body>
</html>
