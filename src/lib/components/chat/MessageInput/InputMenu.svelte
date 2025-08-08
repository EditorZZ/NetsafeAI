<script lang="ts">
	import { DropdownMenu } from 'bits-ui';
	import { flyAndScale } from '$lib/utils/transitions';
	import { getContext, onMount, tick } from 'svelte';

	import { config, user, tools as _tools, mobile } from '$lib/stores';
	import { createPicker } from '$lib/utils/google-drive-picker';

	import { getTools } from '$lib/apis/tools';

	import Dropdown from '$lib/components/common/Dropdown.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import DocumentArrowUpSolid from '$lib/components/icons/DocumentArrowUpSolid.svelte';
	import Switch from '$lib/components/common/Switch.svelte';
	import GlobeAltSolid from '$lib/components/icons/GlobeAltSolid.svelte';
	import WrenchSolid from '$lib/components/icons/WrenchSolid.svelte';
	import CameraSolid from '$lib/components/icons/CameraSolid.svelte';
	import PhotoSolid from '$lib/components/icons/PhotoSolid.svelte';
	import CommandLineSolid from '$lib/components/icons/CommandLineSolid.svelte';

	const i18n = getContext('i18n');

	export let selectedToolIds: string[] = [];

	export let selectedModels: string[] = [];
	export let fileUploadCapableModels: string[] = [];

	export let screenCaptureHandler: Function;
	export let uploadFilesHandler: Function;
	export let inputFilesHandler: Function;

	export let uploadGoogleDriveHandler: Function;
	export let uploadOneDriveHandler: Function;

	export let onClose: Function;

	let tools = {};
	let show = false;
	let showAllTools = false;

	$: if (show) {
		init();
	}

	let fileUploadEnabled = true;
	$: fileUploadEnabled =
		fileUploadCapableModels.length === selectedModels.length &&
		($user?.role === 'admin' || $user?.permissions?.chat?.file_upload);

	const init = async () => {
		if ($_tools === null) {
			await _tools.set(await getTools(localStorage.token));
		}

		tools = $_tools.reduce((a, tool, i, arr) => {
			a[tool.id] = {
				name: tool.name,
				description: tool.meta.description,
				enabled: selectedToolIds.includes(tool.id)
			};
			return a;
		}, {});
	};

	const detectMobile = () => {
		const userAgent = navigator.userAgent || navigator.vendor || window.opera;
		return /android|iphone|ipad|ipod|windows phone/i.test(userAgent);
	};

	function handleFileChange(event) {
		const inputFiles = Array.from(event.target?.files);
		if (inputFiles && inputFiles.length > 0) {
			console.log(inputFiles);
			inputFilesHandler(inputFiles);
		}
	}
</script>

<!-- Hidden file input used to open the camera on mobile -->
<input
	id="camera-input"
	type="file"
	accept="image/*"
	capture="environment"
	on:change={handleFileChange}
	style="display: none;"
/>

<Dropdown
	bind:show
	on:change={(e) => {
		if (e.detail === false) {
			onClose();
		}
	}}
>
	<!-- Sichtbare Buttons -->

	<div class="flex gap-3 my-2">
		<button
			class="px-2 @xl:px-2.5 py-2 flex items-center justify-center gap-1.5 text-sm transition-colors duration-300 max-w-full overflow-hidden hover:bg-gray-50 dark:hover:bg-gray-800 bg-transparent text-gray-600 dark:text-gray-300 focus:outline-hidden rounded-full"
			on:click={() => screenCaptureHandler()}
			type="button"
		>
			<CameraSolid />
			<span class="btn-label font-medium">{$i18n.t('Capture')}</span>
		</button>

		<button
			class="px-2 @xl:px-2.5 py-2 flex items-center justify-center gap-1.5 text-sm transition-colors duration-300 max-w-full overflow-hidden hover:bg-gray-50 dark:hover:bg-gray-800 bg-transparent text-gray-600 dark:text-gray-300 focus:outline-hidden rounded-full"
			on:click={() => uploadFilesHandler()}
			type="button"
		>
			<DocumentArrowUpSolid />
			<span class="btn-label font-medium">{$i18n.t('Upload Files')}</span>
		</button>
	</div>

	<style>
	@media (max-width: 400px) {
		.btn-label {
			display: none !important;
		}
	}
	</style>

	<div slot="content">
	</div>
</Dropdown>
