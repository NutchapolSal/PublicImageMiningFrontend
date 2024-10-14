<script lang="ts">
	import { Client } from '@gradio/client';

	let video: HTMLVideoElement;
	let canvas: HTMLCanvasElement;
	let startbutton: HTMLButtonElement;
	let mediaDevices: MediaDeviceInfo[] = $state([]);
	let deviceId: string | null = $state(null);
	let app: Client | null = $state(null);
	let loginFailure = $state(false);
	let username = $state('');
	let password = $state('');
	let serverUrl = $state('');
	let loginUsername = $state('');
	let lastResult: { modelGuess: string; output: string } | null = $state(null);
	let cameraGranted = $state(false);

	async function allowWebcam() {
		await navigator.mediaDevices.getUserMedia({ video: true, audio: false });

		cameraGranted = true;
	}

	$effect(() => {
		if (cameraGranted) {
			navigator.mediaDevices.enumerateDevices().then((devices) => {
				mediaDevices = devices.filter((d) => d.kind === 'videoinput');
			});
		}
	});

	$effect(() => {
		if (deviceId) {
			navigator.mediaDevices
				.getUserMedia({ video: { deviceId: deviceId }, audio: false })
				.then((stream) => {
					video.srcObject = stream;
					video.play();
				})
				.catch((err) => {
					console.log('An error occurred: ' + err);
				});
		}
	});

	async function login() {
		loginFailure = false;
		try {
			app = await Client.connect(serverUrl, { auth: [username, password] });
			loginUsername = username;
		} catch (error) {
			loginFailure = true;
			throw error;
		}
	}

	async function predict_and_save_image(image: Blob) {
		if (!app) {
			return;
		}
		const result = await app.predict('/predict_and_save_image', {
			image: image,
			tag: 'none',
			username: loginUsername
		});
		lastResult = {
			modelGuess: (result.data as unknown[])[0] as string,
			output: (result.data as unknown[])[1] as string
		};
	}
	}

	$effect(() => {
		navigator.permissions.query({ name: 'camera' }).then((permissionStatus) => {
			cameraGranted = permissionStatus.state === 'granted';
			permissionStatus.onchange = () => {
				cameraGranted = permissionStatus.state === 'granted';
			};
		});

		startbutton.addEventListener(
			'click',
			(ev) => {
				takepicture();
				ev.preventDefault();
			},
			false
		);

		const takepicture = () => {
			const context = canvas.getContext('2d');
			if (context && video.videoWidth && video.videoHeight) {
				canvas.width = video.videoWidth;
				canvas.height = video.videoHeight;
				context.drawImage(video, 0, 0, video.videoWidth, video.videoHeight);

				canvas.toBlob((blob) => {
					if (blob) {
						predict_and_save_image(blob);
					}
				});
			} else {
				clearphoto();
			}
		};

		const clearphoto = () => {
			const context = canvas.getContext('2d');
			if (context) {
				context.fillStyle = '#AAA';
				context.fillRect(0, 0, canvas.width, canvas.height);
			}
		};
	});
</script>

<form>
	<input type="url" bind:value={serverUrl} placeholder="serverUrl" />
	<input type="text" bind:value={username} placeholder="username" />
	<input type="password" bind:value={password} placeholder="password" />
	<button type="button" onclick={login}>login</button>
	{#if app}
		✅
	{/if}
	{#if loginFailure}
		❌
	{/if}
</form>
{#if !cameraGranted}
	<button onclick={allowWebcam}>allow access to webcam</button>
{:else}
	<label>
		📷
		<select bind:value={deviceId}>
			{#each mediaDevices as d}
				<option value={d.deviceId}>{d.label}</option>
			{/each}
		</select>
	</label>
{/if}
<br />
<video bind:this={video} playsinline></video>
<canvas bind:this={canvas}></canvas>
<br />
<button bind:this={startbutton}>capture</button>
<br />
{#if lastResult}
	<p>Model guess: {lastResult.modelGuess}</p>
	<p>Output: {lastResult.output}</p>
{/if}

<style>
	video {
		width: 20%;
		height: auto;
	}
	canvas {
		width: 10%;
		height: auto;
	}
</style>
