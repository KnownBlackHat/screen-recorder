<script>
let videoStream = null;
let downloadUrl = null;
let audioStream = null;
let chunks = [];
let recorder = {state: 'inactive'};
let micPermission = false;
let videoPermission = false;

const setVideoFeedback = async () => {
    if (videoStream) {
        const video = document.querySelector('video');
        video.srcObject = videoStream;
        video.controls = false;
        video.play();
    }
}

const startRecording = async () => {
    if (micPermission || videoPermission) {
        let mixedStream;
        if (videoPermission && micPermission) {
        mixedStream = new MediaStream([...videoStream.getTracks(), ...audioStream.getTracks()]);
        }
        else if (videoPermission) {
        mixedStream = new MediaStream([...videoStream.getTracks()]);
        }
        else if (micPermission) {
        mixedStream = new MediaStream([...audioStream.getTracks()]);
        }
        recorder = new MediaRecorder(mixedStream, { mimeType: 'video/webm;codec=vp9' });
        recorder.ondataavailable = e => chunks.push(e.data);
        recorder.onstop = ()  => {
            const videoBlob = new Blob(chunks, {type: 'video/mp4'});
            chunks = [];
            downloadUrl = URL.createObjectURL(videoBlob);
            const video = document.querySelector('video');
            video.src = downloadUrl;
            video.controls = true;
            video.srcObject = null;
            video.load();
            video.onload(() => {
                    video.play();
                });
            videoStream.getTracks().forEach(track => track.stop());
            audioStream.getTracks().forEach(track => track.stop());
            }
        recorder.start(200);
        downloadUrl = null;
        }
    }


</script>

<svelte:head>
 <title>Screen Recorder</title>
</svelte:head>

<div class="bg-gray-800 text-white font-light text-center text-xl mb-4 rounded uppercase"> Video Recorder </div>

<video controls autoplay class="video-feedback border-2 border-gray-900 w-full max-h-96 rounded">
    <track kind="captions">
</video>

{#if recorder.state !== "recording" && recorder.state !== "paused"}
<div class="text-center rounded-full m-4">
<button class="bg-{videoPermission ? 'green': 'red' }-500 p-2 text-4xl rounded-full hover:opacity-90"
on:click={async () => {
    if (videoPermission) {
        videoStream = null;
        videoPermission = false;
        document.querySelector('video').srcObject = null;
    } else {
    try {
    videoStream = await navigator.mediaDevices.getDisplayMedia({video: true, audio: true})
    videoPermission = true;
    await setVideoFeedback();
} catch (err) {
    videoPermission = false;
    }}
}}>📺</button>
<button class="bg-{micPermission ? 'green': 'red' }-500 p-2 text-4xl rounded-full hover:opacity-90"
on:click={async () => {
    if (micPermission) {
        audioStream = null;
        micPermission = false;
    } else {
    try {
    audioStream = await navigator.mediaDevices.getUserMedia({audio: {
            echoCancellation: true,
            noiseSuppression: true
            }})
    micPermission = true;
    } catch (err) {
    micPermission = false;
    }}
    }}>🎙️</button>
</div>
{/if}
<div class="flex items-center text-white justify-center space-x-4 mt-4">

{#if recorder.state === 'recording'}
<button class="bg-yellow-500 p-4 rounded-full hover:opacity-90 w-full"
on:click={() => {recorder.pause(); recorder.x = 1}}>Pause</button>
<button class="bg-red-500 p-4 rounded-full hover:opacity-90 w-full"
on:click={() => recorder.stop()}>Stop</button>

{:else if recorder.state === 'paused'}
<button class="bg-green-500 p-4 rounded-full hover:opacity-90 w-full"
on:click={() => {recorder.resume(); recorder.x = 0}}>Resume</button>
<button class="bg-red-500 p-4 rounded-full hover:opacity-90 w-full"
on:click={() => recorder.stop()}>Stop</button>

{:else if micPermission || videoPermission}
<button class="bg-green-500 p-4 rounded-full hover:opacity-90 w-full"
on:click={startRecording}>Start</button>
{/if}
</div>

{#if downloadUrl}
    <div class="flex justify-center mt-4 text-white items-center text-center"> <a class="download bg-green-500 p-4 rounded-full hover:opacity-90 w-full" href={downloadUrl} download="video" >Download</a></div>
{/if}


