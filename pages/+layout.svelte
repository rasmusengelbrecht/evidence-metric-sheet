<script>
	import '@evidence-dev/tailwind/fonts.css';
	import '../app.css';
	import { EvidenceDefaultLayout } from '@evidence-dev/core-components';
	export let data;
</script>

<EvidenceDefaultLayout {data} logo="/image.webp" hideSidebar=true hideBreadcrumbs=true>
	<slot slot="content" />
</EvidenceDefaultLayout>
