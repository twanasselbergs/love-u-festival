<script>
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';
	import { X } from 'lucide-svelte';
	import artists from '$lib/data/artists.json';
	import { t } from 'svelte-i18n';
	import { browser } from '$app/environment';
	import { assets } from '$app/paths';

	const stages = ['Poton', 'The Lake', 'The Club', 'Hangar'];

	const slotWidth = 60;
	const rowHeight = 80;
	const timeColWidth = 100;

	const timeSlots = [];
	for (let hour = 10; hour <= 23; hour++) {
		for (let min = 0; min < 60; min += 15) {
			const h = hour.toString().padStart(2, '0');
			const m = min.toString().padStart(2, '0');
			timeSlots.push(`${h}:${m}`);
		}
	}

	let selectedDay = 'sat';
	let selectedGenre = 'All';

	function timeToMinutes(time) {
		const [h, m] = time.split(':').map(Number);
		return h * 60 + m;
	}

	function getHorizontalArtistStyle(artist) {
		const stageIdx = stages.indexOf(artist.stage);
		const startMinutes = timeToMinutes(artist.start);
		const endMinutes = timeToMinutes(artist.end);
		const firstSlotMinutes = timeToMinutes(timeSlots[0]);

		if (stageIdx === -1 || isNaN(startMinutes) || isNaN(endMinutes)) return 'display:none';

		const top = stageIdx * rowHeight;
		const left = ((startMinutes - firstSlotMinutes) / 15) * slotWidth;
		const width = ((endMinutes - startMinutes) / 15) * slotWidth;
		const minWidth = 40;

		return `
      top: ${top}px;
      left: ${left}px;
      width: ${Math.max(width, minWidth)}px;
      height: ${rowHeight - 10}px;
      opacity: 0.95;
    `;
	}

	function getStageBg(stage) {
		switch (stage) {
			case 'Poton':
				return 'bg-vermilion';
			case 'The Lake':
				return 'bg-cerulean';
			case 'The Club':
				return 'bg-saffron';
			case 'Hangar':
				return 'bg-purple-600';
			default:
				return '';
		}
	}

	$: genres = ['All', ...new Set(artists.map((a) => a.genre).filter((g) => g && g !== '?'))];

	$: filteredArtists = artists.filter(
		(a) => a.day === selectedDay && (selectedGenre === 'All' || a.genre === selectedGenre)
	);

	let selectedArtist = null;

	const openModal = (artist) => {
		selectedArtist = artist;
	};

	const closeModal = () => {
		selectedArtist = null;
	};

	function formatDate(date) {
		const options = { month: 'long', day: 'numeric' };
		return date.toLocaleDateString(undefined, options);
	}
	const today = new Date();
	$: todayString = `${formatDate(today)}`;

	let favoritedArtists = new Set();

	onMount(() => {
		if (browser) {
			const stored = localStorage.getItem('favoritedArtists');
			if (stored) {
				try {
					const parsed = JSON.parse(stored);
					if (Array.isArray(parsed)) {
						favoritedArtists = new Set(parsed);
					}
				} catch (e) {
					console.error('Error');
				}
			}
		}
	});

	function toggleFavorite(artistName) {
		if (favoritedArtists.has(artistName)) {
			favoritedArtists.delete(artistName);
		} else {
			favoritedArtists.add(artistName);
		}
		if (browser) {
			localStorage.setItem('favoritedArtists', JSON.stringify([...favoritedArtists]));
		}
		favoritedArtists = new Set(favoritedArtists);
	}

	$: if (browser) {
		if (selectedArtist) {
			document.body.style.overflow = 'hidden';
		} else {
			document.body.style.overflow = '';
		}
	}
</script>

<main class="pb-30 flex flex-col gap-6 px-6 pt-20">
	<section class="flex w-full justify-between gap-4">
		<div class="bg-vermilion w-50 flex items-center justify-center gap-1.5 rounded-full p-1.5">
			<span class="material-icons-round text-white">location_on</span>
			<p class="mr-1 text-sm font-extrabold text-white">Strijkviertel, Utrecht</p>
		</div>
		<img
			src="{assets}/img/logos/logo-white-transparent.png"
			alt="Logo"
			class="h-10 w-10 rounded-full object-cover pl-0.5" />
	</section>

	<section class="flex w-full flex-col gap-1">
		<p class="text-sm text-gray-400">{todayString}</p>
		<h1 class="sansation-bold text-xl text-gray-800 dark:text-white/90">{$t('events')}</h1>
	</section>

	<section class="flex gap-4">
		<button
			class="sansation-bold rounded-full px-4 py-2 text-sm font-bold text-white transition-colors duration-500"
			class:bg-vermilion={selectedDay === 'sat'}
			class:text-white={selectedDay === 'sat'}
			class:dark:text-gray-500={selectedDay === 'sun'}
			on:click={() => (selectedDay = 'sat')}>
			{$t('saturday')}
		</button>
		<button
			class="sansation-bold rounded-full px-4 py-2 text-sm font-bold text-white transition-colors duration-500"
			class:bg-vermilion={selectedDay === 'sun'}
			class:text-white={selectedDay === 'sun'}
			class:dark:text-gray-500={selectedDay === 'sat'}
			on:click={() => (selectedDay = 'sun')}>
			{$t('sunday')}
		</button>
	</section>

	<section class="scrollbar-none -mx-6 flex gap-2 overflow-hidden overflow-x-auto px-6">
		{#each genres as genre}
			<button
				class="sansation-bold dark:bg-gray3 rounded-full px-4 py-1 text-sm font-bold transition-colors duration-500"
				class:bg-black={selectedGenre === genre}
				class:text-white={selectedGenre === genre}
				class:bg-gray-200={selectedGenre !== genre}
				class:text-gray-400={selectedGenre !== genre}
				on:click={() => (selectedGenre = genre)}>
				{genre}
			</button>
		{/each}
	</section>

	<section class="scrollbar-none -mx-6 overflow-x-auto overflow-y-hidden px-6">
		<div
			class="relative"
			style={`height: ${stages.length * rowHeight + 60}px; min-width: ${timeSlots.length * slotWidth + 80}px`}>
			<div class="flex">
				<div style={`width: ${timeColWidth}px`}></div>
				{#each timeSlots as time}
					<div
						class="border-b border-gray-300 bg-gray-100 py-2 text-center text-xs font-bold dark:bg-[#1d1d1e] dark:text-gray-400"
						style={`width: ${slotWidth}px`}>
						{time}
					</div>
				{/each}
			</div>

			<div class="top-8.5 absolute -left-8 flex flex-col">
				{#each stages as stage, i}
					<div
						class="sansation-bold flex items-center justify-end border-r border-gray-400/15 bg-gray-100 pr-2 text-sm font-semibold dark:bg-[#1d1d1e] dark:text-gray-400"
						style={`height: ${rowHeight}px; width: ${timeColWidth}px; position: sticky; left: 0; z-index: 20;`}>
						{stage}
						<span class={`ml-1 h-2 w-2 rounded-full ${getStageBg(stage)}`}></span>
					</div>
				{/each}
			</div>

			<div class="absolute left-[95px] right-0 top-9 z-0">
				{#each stages as _, i}
					<div
						class="border-t border-gray-400/15"
						style={`top: ${i * rowHeight + rowHeight}px; height: 0; position: absolute; width: 100%;`}>
					</div>
				{/each}
			</div>

			<div class="absolute top-10 ml-5" style={`left: ${timeColWidth}px;`}>
				{#each filteredArtists as artist}
					{#if artist.start && artist.end}
						<div
							on:click={() => openModal(artist)}
							class={`absolute flex cursor-pointer items-center gap-2 rounded-full px-4 text-left text-white ${getStageBg(artist.stage)}`}
							role
							style={getHorizontalArtistStyle(artist)}>
							<img
								src="{assets}{artist.img}"
								alt="Artist"
								class="mb-1 h-10 w-10 rounded-full object-cover" />
							<div class="flex w-full items-center justify-between">
								<div class="flex flex-col">
									<h2 class="sansation-bold text-sm">{artist.artist}</h2>
									<p class="text-xs">{artist.start} {$t('to')} {artist.end}</p>
								</div>
								<button
									type="button"
									on:click|stopPropagation={() => toggleFavorite(artist.artist)}
									class="material-icons-round scale-80 text-white transition-transform duration-300 active:scale-75">
									{favoritedArtists?.has(artist.artist) ? 'favorite' : 'favorite_border'}
								</button>
							</div>
						</div>
					{/if}
				{/each}
			</div>
		</div>
	</section>

	<div
		class="fixed inset-0 z-40 bg-black transition-opacity duration-500 ease-in-out"
		class:opacity-20={selectedArtist}
		class:opacity-0={!selectedArtist}
		class:pointer-events-none={!selectedArtist}>
	</div>
	{#if selectedArtist}
		<div
			class="fixed inset-x-0 bottom-0 z-50 flex items-center justify-center"
			transition:fly={{ y: 650, duration: 500, opacity: 1 }}>
			<div
				class="scrollbar-none dark:bg-gray3 relative flex max-h-[80vh] w-full max-w-md flex-col gap-2 overflow-y-auto rounded-t-3xl bg-white p-5 py-10 shadow dark:text-white/70">
				<button
					on:click={closeModal}
					class="absolute right-3 top-3 rounded-full bg-gray-200 p-1 text-gray-400 dark:bg-gray-700">
					<X class="h-4 w-4 stroke-[4]" />
				</button>
				<div
					class={`mb-2 flex items-center gap-4 rounded-full p-2 px-4 ${getStageBg(selectedArtist.stage)}`}>
					<img
						src="{assets}{selectedArtist.img}"
						alt=""
						class="h-10 w-10 rounded-full object-cover" />
					<div class="flex-col text-white">
						<h2 class="sansation-bold">{selectedArtist.artist}</h2>
						<p class="">{selectedArtist.song}</p>
					</div>
				</div>
				{#if selectedArtist.vid}
					<iframe
						title="YouTube"
						src={selectedArtist.vid}
						frameborder="0"
						class="rounded-4xl mb-2 h-48 w-full shadow">
					</iframe>
				{/if}
				<div class="flex gap-2">
					<span class="material-icons-round text-vermilion">info</span>
					<p class="text-vermilion sansation-bold dark:text-gray">{$t('additional_information')}</p>
				</div>
				<p class="px-1 text-gray-600 dark:text-white/70">{$t(selectedArtist.desc)}</p>
				<div class="my-2 flex items-center gap-2 px-1">
					<span class="material-icons-round text-saffron">watch_later</span>
					<p class="sansation-bold dark:text text-sm text-gray-800 dark:text-white/90">
						{selectedArtist.date}
					</p>
				</div>
				<div class="flex items-center">
					<div class="flex items-center gap-2 px-1">
						<span class="material-icons-round text-saffron">festival</span>
						<p class="sansation-bold text-sm text-gray-800 dark:text-white/90">
							{selectedArtist.stage}
						</p>
					</div>
					<div class="flex items-center gap-2 px-1">
						<span class="material-icons-round text-saffron">music_note</span>
						<p class="sansation-bold text-sm text-gray-800 dark:text-white/90">
							{selectedArtist.genre}
						</p>
					</div>
				</div>
			</div>
		</div>
	{/if}
</main>
