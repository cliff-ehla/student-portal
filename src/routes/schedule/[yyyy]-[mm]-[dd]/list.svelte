<script context="module">
	import {http, onFail} from "$lib/http";

	export const load = async ({fetch, session}) => {
		if (!session.user_info) {
			return {
				status: 302,
				redirect: '/login'
			}
		}
		const {success, data, debug} = await http.post(fetch, '/zoom/zoom_list_all', {})
		if (!success) return onFail(debug)
		return {
			props: {
				zoom_list: data
			}
		}
	}
</script>

<script>
	import dayjs from "dayjs";
	import {page} from "$app/stores";
	import isToday from "dayjs/plugin/isToday.js";
	import isBetween from "dayjs/plugin/isBetween.js";
	import ZoomPreview from '$lib/zoom/zoom-preview.svelte'
	import * as animateScroll from "svelte-scrollto";
	import {browser} from "$app/env";
	import {onMount} from "svelte";

	export let zoom_list
	dayjs.extend(isToday)
	dayjs.extend(isBetween)
	$: date_key = `${$page.params.yyyy}-${$page.params.mm}-${$page.params.dd}`
	$: zoom_list_by_date = date_key ? groupByDate(zoom_list) : []
	$: reload = $page.query.get('reload')
	$: {
		if (browser && (date_key || reload)) {
			setTimeout(scrollToDate, 1)
		}
	}
	onMount(() => {
		scrollToDate()
	})
	const groupByDate = (zoom_list) => {
		let results = []
		zoom_list.forEach(z => {
			if (!z) return
			if (dayjs(z.start_date).isSame(date_key, 'month')) {
				let date = dayjs(z.start_date).format('YYYY-MM-DD')
				let obj = results.find(r => r.date === date)
				if (!obj) {
					results.push({
						date,
						zoom_list: [z]
					})
				} else {
					obj.zoom_list.push(z)
				}
			}
		})
		return results
	}

	const scrollToDate = () => {
		const today_key = dayjs().format('YYYY-MM-DD')
		animateScroll.scrollTo({
			element: `[data-date="${today_key}"]`,
			offset: -48,
			onDone: (el) => {
				if (!el) return
				el.classList.add('bg-yellow-500')
				setTimeout(() => {
					el.classList.remove('bg-yellow-500')
				}, 2000)
			}
		})
	}

</script>

<div class="max-w-screen-lg mx-auto">
	{#if zoom_list_by_date.length}
		{#each zoom_list_by_date as date}
			<div data-date={dayjs(date.date).format('YYYY-MM-DD')} class="transition-colors duration-1000 flex items-start">
				<div class="w-40 px-2 flex items-center">
					<div class:active={dayjs(date.date).isToday()} class="rounded-full w-10 h-10 cc mr-2">{dayjs(date.date).format('D')}</div>
					<p class="text-xs uppercase mt-0.5">{dayjs(date.date).format('MMM, ddd')}</p>
				</div>
				<div class="w-full p-2">
					{#each date.zoom_list as z}
						<ZoomPreview {z}/>
					{/each}
				</div>
			</div>
		{/each}
	{:else}
		<div class="py-12">
			<p class="px-8 text-xl text-gray-300">你在{dayjs(date_key).format('MMM')}没有任何課堂</p>
		</div>
	{/if}
</div>