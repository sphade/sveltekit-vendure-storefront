<script lang="ts">
	import { page } from '$app/stores';
	import AddAddressModal from '$lib/components/modals/AddAddressModal.svelte';
	import EditAddressModal from '$lib/components/modals/EditAddressModal.svelte';
	import { Button } from '$lib/components/ui/button';
	import * as Card from '$lib/components/ui/card';
	import { Badge } from '$lib/components/ui/badge';
	import { Pen, Trash } from 'lucide-svelte';
	import { goto, invalidateAll, preloadData, pushState } from '$app/navigation';
	import { editAddressModalStore } from '$lib/stores/modalStore';
	import { deleteAddress } from '$lib/mutation';
	import * as AlertDialog from '$lib/components/ui/alert-dialog';

	$: {
		if ($page.state?.form) {
			editAddressModalStore.setTrue();
		}
	}
</script>

{#if $page.state.form}
	<EditAddressModal />
{/if}

{#if $page.data.user?.activeCustomer?.addresses?.length === 0}
	<div class="flex h-[50vh] flex-col items-center justify-center text-center">
		<h1 class="font-display text-3xl font-semibold">You haven’t saved any addresses yet</h1>
		<p class="mb-5 text-sm text-muted-foreground">Add a new address easily below</p>
		<AddAddressModal />
	</div>
{:else}
	<div>
		<div class="mb-2 ml-auto w-fit px-3">
			<AddAddressModal />
		</div>

		<div class="mb-3 grid gap-5 lg:grid-cols-2">
			{#each $page.data.user?.activeCustomer?.addresses ?? [] as address (address)}
				<Card.Root>
					<Card.Header class="border-none">
						<Card.Title class="capitalize ">{address.fullName}</Card.Title>
					</Card.Header>
					<Card.Content class="flex justify-between text-sm capitalize text-muted-foreground ">
						<div>
							<p>{address.streetLine1}</p>
							<p>{address.postalCode}</p>
							<p>{address.city}</p>
							<p>{address.country.name}</p>
						</div>
						<div class="mt-auto w-[150px] space-y-2">
							{#if address.defaultShippingAddress}
								<Badge variant="secondary" class="uppercase">Shipping Address</Badge>
							{/if}
							{#if address.defaultBillingAddress}
								<Badge variant="secondary" class=" uppercase">Billing Address</Badge>
							{/if}
						</div>
					</Card.Content>
					<Card.Footer class=" -pb-2 gap-5 pt-1 text-sm">
						<Button
							variant="ghost"
							class="flex items-center gap-2"
							href="/me/addresses/edit/{address.id}"
							on:click={async (e) => {
								const href = `/me/addresses/edit/${address.id}`;
								if (innerWidth < 600 || e.shiftKey || e.metaKey || e.ctrlKey) return;
								e.preventDefault();
								const result = await preloadData(href);
								if (result.type === 'loaded') {
									pushState(href, {
										address: result.data.address,
										form: result.data.form
									});
								} else {
									goto(href);
								}
							}}
						>
							<Pen class="size-3" />
							<p>Edit</p>
						</Button>

						<AlertDialog.Root>
							<AlertDialog.Trigger>
								<Button
									variant="destructive"
									class="gap-2 bg-destructive/20 text-destructive hover:bg-destructive/30 "
								>
									<Trash class="size-3" />
									<p>Remove</p>
								</Button>
							</AlertDialog.Trigger>
							<AlertDialog.Content class="w-full  p-5">
								<AlertDialog.Header class="mt-10">
									<AlertDialog.Title class="font-display text-lg sm:text-xl md:text-3xl  "
										>Are you absolutely sure?</AlertDialog.Title
									>
									<AlertDialog.Description>
										This action cannot be undone. This will permanently delete this address and
										remove it data from our servers.
									</AlertDialog.Description>
								</AlertDialog.Header>
								<AlertDialog.Footer class="mt-5">
									<AlertDialog.Cancel class="w-full">Cancel</AlertDialog.Cancel>
									<AlertDialog.Action
										on:click={async () => {
											await deleteAddress.mutate({ id: address.id });
											await invalidateAll();
										}}
										class=" w-full bg-destructive hover:bg-destructive/90"
									>
										Delete addresses</AlertDialog.Action
									>
								</AlertDialog.Footer>
							</AlertDialog.Content>
						</AlertDialog.Root>
					</Card.Footer>
				</Card.Root>
			{/each}
		</div>
	</div>
{/if}
