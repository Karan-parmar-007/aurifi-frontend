<script lang="ts">
	import { Checkbox, Input, Label, Modal, Select, Button } from 'flowbite-svelte';
	import TagInput from '$lib/components/TagInput.svelte';
	import { systemColumnModal } from '../../store/toogleModal.svelte';
	import { VITE_API_URL } from '$lib/constants';

	interface Tag {
		id: string;
		text: string;
	}

	// Define props using $props()
	const { editingColumn = null, fetchData = async () => {} } = $props<{
		editingColumn?: any;
		fetchData: () => Promise<void>;
	}>();

	let isLoading = $state(false);
	let errorMessage = $state('');
	let tags = $state<Tag[]>([]);
	let editTags = $state<Tag[]>([]);

	// Form data for adding a new column
	let formData = $state({
		column_name: '',
		description: '',
		alt_names: [],
		asset_class: '',
		datatype: '',
		general_mandatory: false,
		is_currency: false
	});

	// Form data for editing an existing column
	let editFormData = $state({
		column_name: '',
		description: '',
		alt_names: [],
		asset_class: '',
		datatype: '',
		general_mandatory: false,
		is_currency: false
	});

	// Reactive variables for binding form inputs
	let columnName = $state('');
	let description = $state('');
	let assetClass = $state('');
	let datatype = $state('');
	let generalMandatory = $state(false);
	let isCurrency = $state(false);

	// Options for dropdowns
	let assetClassOptions = [
		{ value: 'Real Estate', name: 'Real Estate' },
		{ value: 'Vehicles', name: 'Vehicles' },
		{ value: 'Securities', name: 'Securities' }
	];

	let dataTypeOptions = [
		{ value: 'currency', name: 'currency' },
		{ value: 'string', name: 'string' },
		{ value: 'number', name: 'number' },
		{ value: 'date', name: 'date' },
		{ value: 'boolean', name: 'boolean' }
	];

	// Initialize form data when modal opens
	if (systemColumnModal.isSystemColumnModalOpen) {
		if (editingColumn) {
			// Populate edit form data
			editFormData = {
				column_name: editingColumn.column_name,
				description: editingColumn.description,
				alt_names: Array.isArray(editingColumn.alt_names)
					? editingColumn.alt_names
					: [editingColumn.alt_names],
				asset_class: editingColumn.asset_class,
				datatype: editingColumn.datatype,
				general_mandatory: editingColumn.general_mandatory,
				is_currency: editingColumn.is_currency
			};
			editTags = createTagsFromAltNames(editingColumn.alt_names);

			// Set reactive variables for binding
			columnName = editFormData.column_name;
			description = editFormData.description;
			assetClass = editFormData.asset_class;
			datatype = editFormData.datatype;
			generalMandatory = editFormData.general_mandatory;
			isCurrency = editFormData.is_currency;
		} else {
			// Reset add form data
			formData = {
				column_name: '',
				description: '',
				alt_names: [],
				asset_class: '',
				datatype: '',
				general_mandatory: false,
				is_currency: false
			};
			tags = [];

			// Reset reactive variables for binding
			columnName = '';
			description = '';
			assetClass = '';
			datatype = '';
			generalMandatory = false;
			isCurrency = false;
		}
	}

	// Sync reactive variables back to the appropriate form data
	if (editingColumn) {
		editFormData.column_name = columnName;
		editFormData.description = description;
		editFormData.asset_class = assetClass;
		editFormData.datatype = datatype;
		editFormData.general_mandatory = generalMandatory;
		editFormData.is_currency = isCurrency;
	} else {
		formData.column_name = columnName;
		formData.description = description;
		formData.asset_class = assetClass;
		formData.datatype = datatype;
		formData.general_mandatory = generalMandatory;
		formData.is_currency = isCurrency;
	}

	function handleTagsChange(newTags: Tag[]) {
		tags = newTags;
		formData.alt_names = newTags.map((tag) => tag.text);
	}

	function handleEditTagsChange(newTags: Tag[]) {
		editTags = newTags;
		editFormData.alt_names = newTags.map((tag) => tag.text);
	}

	function createTagsFromAltNames(altNames: string | string[]): Tag[] {
		if (!altNames) {
			return [];
		}

		if (typeof altNames === 'string') {
			try {
				const names = JSON.parse(altNames);
				if (Array.isArray(names)) {
					return names.map((name) => ({
						id: Date.now().toString(36) + Math.random().toString(36).substring(2),
						text: name
					}));
				}
				return altNames.split(',').map((name) => ({
					id: Date.now().toString(36) + Math.random().toString(36).substring(2),
					text: name.trim()
				}));
			} catch (e) {
				return [
					{
						id: Date.now().toString(36) + Math.random().toString(36).substring(2),
						text: altNames
					}
				];
			}
		}

		if (Array.isArray(altNames)) {
			return altNames.map((name) => ({
				id: Date.now().toString(36) + Math.random().toString(36).substring(2),
				text: name
			}));
		}

		return [];
	}

	// Check if column name already exists (for adding new column)
	async function isColumnNameUnique(columnName: string): Promise<boolean> {
		try {
			const response = await fetch(`${VITE_API_URL}/admin/get_system_columns`);
			if (!response.ok) {
				throw new Error('Failed to fetch columns');
			}
			const data = await response.json();
			if (!data?.data || !data.data.length) return true;
			return !data.data.some(
				(column) => column.column_name.toLowerCase() === columnName.toLowerCase()
			);
		} catch (error) {
			console.error('Error checking column name uniqueness:', error);
			return false;
		}
	}

	async function handleSubmit(e) {
		e.preventDefault();
		isLoading = true;
		errorMessage = '';

		if (/\s/.test(formData.column_name)) {
			errorMessage = 'Column name cannot contain spaces or whitespace characters.';
			isLoading = false;
			return;
		}

		if (!(await isColumnNameUnique(formData.column_name))) {
			errorMessage = `Column name "${formData.column_name}" already exists. Please use a unique name.`;
			isLoading = false;
			return;
		}

		try {
			const dataToSubmit = {
				...formData,
				alt_names: tags.map((tag) => tag.text)
			};

			const response = await fetch(`${VITE_API_URL}/admin/add_system_column`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify(dataToSubmit)
			});

			if (!response.ok) {
				console.log('error uploading column data:', response);
				errorMessage = 'Failed to add column. Please try again.';
				isLoading = false;
				return;
			}

			const result = await response.json();
			console.log(result);
			await fetchData();
			systemColumnModal.isSystemColumnModalOpen = false;
		} catch (error) {
			console.log(error.message);
			errorMessage = `Error: ${error.message}`;
			isLoading = false;
		}
	}

	async function saveEdit(id: string) {
		isLoading = true;
		errorMessage = '';

		if (/\s/.test(editFormData.column_name)) {
			errorMessage = 'Column name cannot contain spaces or whitespace characters.';
			isLoading = false;
			return;
		}

		try {
			const dataToSubmit = {
				original_column_name: editingColumn.column_name,
				...editFormData,
				alt_names: editTags.map((tag) => tag.text)
			};

			const response = await fetch(
				`${VITE_API_URL}/admin/update_system_column/${id}`,
				{
					method: 'PUT',
					headers: {
						'Content-Type': 'application/json'
					},
					body: JSON.stringify(dataToSubmit)
				}
			);

			if (!response.ok) {
				const errorData = await response.json();
				console.log(errorData);
				errorMessage = 'Failed to update column. Please try again.';
				isLoading = false;
				return;
			}

			const result = await response.json();
			console.log(result);
			await fetchData();
			systemColumnModal.isSystemColumnModalOpen = false;
		} catch (error) {
			console.log(error.message);
			errorMessage = `Error: ${error.message}`;
			isLoading = false;
		}
	}
</script>

<Modal
	bind:open={systemColumnModal.isSystemColumnModalOpen}
	title={editingColumn ? 'Edit Column' : 'Add Column'}
>
	{#if isLoading}
		<div class="absolute inset-0 z-30 flex h-full w-full items-center justify-center">
			<div
				class="z-40 h-8 w-8 animate-spin rounded-full border-4 border-blue-500 border-t-transparent"
			></div>
		</div>
	{/if}

	<form
		onsubmit={(e) => (editingColumn ? saveEdit(editingColumn._id) : handleSubmit(e))}
		class="flex h-full w-full flex-col items-center gap-4 p-2"
	>
		{#if errorMessage}
			<div class="error-message mb-4 rounded bg-red-100 p-3 text-red-700">
				{errorMessage}
			</div>
		{/if}

		<span class="flex w-full items-center gap-4">
			<span class="flex flex-1 flex-col gap-2">
				<Label for="name">Column Name</Label>
				<Input type="text" name="name" placeholder="Enter Name" bind:value={columnName} required />
			</span>
			<span class="flex flex-1 flex-col gap-2">
				<Label for="desc">Description</Label>
				<Input
					type="text"
					name="desc"
					placeholder="Enter Description"
					bind:value={description}
					required
				/>
			</span>
		</span>
		<span class="flex w-full flex-1 flex-col gap-2">
			<Label for="alt_name">Alt Names</Label>
			<TagInput
				tags={editingColumn ? editTags : tags}
				placeholder="Enter Names"
				onTagsChange={editingColumn ? handleEditTagsChange : handleTagsChange}
				name="alt_name"
			/>
		</span>
		<span class="flex w-full items-center gap-4">
			<span class="flex flex-1 flex-col gap-2">
				<Label for="asset_class">Asset Class</Label>
				<Select items={assetClassOptions} bind:value={assetClass} />
			</span>
			<span class="flex flex-1 flex-col gap-2">
				<Label for="datatype">DataType</Label>
				<Select items={dataTypeOptions} bind:value={datatype} />
			</span>
			<span class="flex h-full flex-1 flex-col justify-end gap-4">
				<span class="mx-auto flex w-[70%] flex-col">
					<span class="flex items-center justify-between">
						<Label for="mandatory">Mandatory</Label>
						<Checkbox name="mandatory" bind:checked={generalMandatory} />
					</span>
					<span class="flex items-center justify-between">
						<Label for="is_currency">Is Currency</Label>
						<Checkbox name="is_currency" bind:checked={isCurrency} />
					</span>
				</span>
			</span>
		</span>
		<div class="mt-4 flex items-center justify-center gap-4">
			<Button
				color="dark"
				type="button"
				onclick={() => (systemColumnModal.isSystemColumnModalOpen = false)}
			>
				Cancel
			</Button>
			<Button color="dark" type="submit">
				{editingColumn ? 'Save' : 'Submit'}
			</Button>
		</div>
	</form>
</Modal>
