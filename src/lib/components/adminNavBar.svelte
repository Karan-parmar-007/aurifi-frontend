<script>
	import { goto } from '$app/navigation';
	import { page } from '$app/state';
	import { Dropdown, DropdownItem, Button } from 'flowbite-svelte';
	import { systemColumnModal } from '../../store/toogleModal.svelte';
	import AdminAddColumnForm from './AdminAddColumnForm.svelte';

	const urlParts = $derived(page.url.pathname.split('/'));
	let componentSelection = $derived(urlParts[2]);
	let activeUrl = $derived(urlParts[2]);
</script>

{#if systemColumnModal.isSystemColumnModalOpen}
	<AdminAddColumnForm />
{/if}

<div class="w-full bg-[#202020]">
	<nav class="flex h-[70px] w-full items-center justify-between overflow-hidden px-6">
		<span class="logo overflow-hidden"
			><img src="/aurifi_logo.png" alt="aurifi logo" width="90" height="40" /></span
		>
		<span class="flex items-center justify-between gap-6 text-white">
			<button
				class="cursor-pointer"
				onclick={() => {
					goto('/admin/user_settings');
				}}>User Settings</button
			>
			<button class="cursor-pointer">System Columns</button>
			<Dropdown {activeUrl} activeClass="bg-[#242C3E33] rounded-xl mx-1 my-1" class="w-40 ">
				<DropdownItem
					class="mx-1 rounded-xl hover:bg-gray-200"
					onclick={() => (componentSelection = 'DebtSheet')}
					href="/admin/DebtSheet">DebtSheet</DropdownItem
				>
				<DropdownItem
					class="mx-1 rounded-xl hover:bg-gray-200"
					onclick={() => (componentSelection = 'Transactions')}
					href="/admin/Transactions">Transactions</DropdownItem
				>
			</Dropdown>
		</span>
	</nav>
</div>
{#if urlParts[2] === 'DebtSheet'}
	<div class="mt-20 flex w-full flex-col px-20">
		<div class="text-[11px] font-semibold text-[#A1A1A1]">System Column</div>
		<div class="flex items-center justify-between">
			<h2 class="text-3xl font-semibold">Book Debt</h2>
			<Button
				color="dark"
				class="text-nowrap"
				onclick={() => {
					systemColumnModal.isSystemColumnModalOpen = true;
				}}>+ Add Columns</Button
			>
		</div>
	</div>
{:else if urlParts[2] === 'Transactions'}
	<div class="mt-20 flex w-full flex-col px-20">
		<div class="text-[11px] font-semibold text-[#A1A1A1]">System Column</div>
		<div class="flex items-center justify-between">
			<h2 class="text-3xl font-semibold">Transaction</h2>
			<Button
				color="dark"
				class="text-nowrap"
				onclick={() => {
					systemColumnModal.isSystemColumnModalOpen = true;
				}}>+ Add Columns</Button
			>
		</div>
	</div>
{:else}
	<div class=""></div>
{/if}

<!-- <div class="absolute w-full px-8 py-4">
	<Navbar>
		<NavBrand class="px-6" href="/">
			<h1 class="text-2xl font-bold text-black dark:text-white">Book Debt Management</h1>
		</NavBrand>
		<div class="flex items-center gap-4 md:order-2">
			<a
				href="/admin/user_settings"
				class={`transition-all duration-1000 hover:text-gray-800 ${pageURL === '/admin/user_settings' ? 'text-gray-600' : ''}`}
				><p>User Settings</p></a
			>
			<a
				href="/admin"
				class={` transition-all duration-1000 hover:text-gray-800 ${pageURL === '/admin' ? 'text-lg' : ''}`}
				><p>System Columns</p></a
			>
			<DarkMode />
		</div></Navbar
	>
</div> -->

<!-- <Modal bind:open={modalOpen} size="xs" autoclose={false} class="w-full">
	<Import />
</Modal> -->
