<script lang="ts">
	type Sign = {
		name: string;
		symbol: string;
		element: '火' | '地' | '風' | '水';
		from: [number, number]; // [月, 日] この日から
	};

	// 星座（開始日の順。山羊座は年をまたぐので最後に置き、該当なしのときの既定にする）
	const signs: Sign[] = [
		{ name: '水瓶座', symbol: '♒', element: '風', from: [1, 20] },
		{ name: '魚座', symbol: '♓', element: '水', from: [2, 19] },
		{ name: '牡羊座', symbol: '♈', element: '火', from: [3, 21] },
		{ name: '牡牛座', symbol: '♉', element: '地', from: [4, 20] },
		{ name: '双子座', symbol: '♊', element: '風', from: [5, 21] },
		{ name: '蟹座', symbol: '♋', element: '水', from: [6, 22] },
		{ name: '獅子座', symbol: '♌', element: '火', from: [7, 23] },
		{ name: '乙女座', symbol: '♍', element: '地', from: [8, 23] },
		{ name: '天秤座', symbol: '♎', element: '風', from: [9, 23] },
		{ name: '蠍座', symbol: '♏', element: '水', from: [10, 24] },
		{ name: '射手座', symbol: '♐', element: '火', from: [11, 23] },
		{ name: '山羊座', symbol: '♑', element: '地', from: [12, 22] }
	];

	const elementTone: Record<Sign['element'], string> = {
		火: '#b3261e',
		地: '#3f6d4e',
		風: '#c9a227',
		水: '#2f5f9e'
	};

	const messages = {
		total: [
			'思いきって動くほど流れに乗れる日。迷ったら「やってみる」を選んで。',
			'足元を整えると運が開けます。机の上を片づけるところから始めて。',
			'人とのつながりが追い風に。久しぶりの相手に連絡してみると吉。',
			'直感が冴える一日。最初に浮かんだアイデアを大切にして。',
			'焦らず一歩ずつが正解。小さな達成を積み重ねると夕方に手応えが。',
			'ふだんと違う道を選ぶと、思わぬ発見がありそう。',
			'聞き役に回ると信頼が深まります。相手の話を最後まで聞いて。',
			'計画の見直しに向く日。やめることを1つ決めると身軽になります。'
		],
		love: [
			'さりげない気づかいが好印象に。',
			'素直な一言が距離を縮めます。',
			'今日は自分を大切にする時間を優先して。',
			'共通の話題から会話が弾みそう。',
			'返事を急がず、ゆっくり育てる日。'
		],
		work: [
			'段取り力が評価される日。',
			'細かい確認がミスを防ぎます。',
			'新しい提案が通りやすい流れ。',
			'午前中に集中すると成果が出ます。',
			'人に頼ることで仕事が早く進みます。'
		],
		money: [
			'衝動買いは一晩おいてから。',
			'小さな節約が後で効いてきます。',
			'自分への投資にツキあり。',
			'財布の中を整理すると吉。',
			'思わぬところで得をしそう。'
		],
		color: ['ネイビー', 'ミントグリーン', 'コーラルピンク', 'マスタード', 'ラベンダー', 'ホワイト', 'テラコッタ', 'スカイブルー'],
		item: ['ハンカチ', '手帳', '温かいお茶', 'イヤホン', '観葉植物', '万年筆', 'お気に入りのマグ', '折りたたみ傘']
	};

	let birthday = $state('');
	let today = $state(new Date());

	function signOf(month: number, day: number): Sign {
		let found = signs[signs.length - 1]; // 1/1〜1/19 は山羊座
		for (const s of signs) {
			const [m, d] = s.from;
			if (month > m || (month === m && day >= d)) found = s;
		}
		return found;
	}

	// 日付と星座から決まる疑似乱数（同じ日・同じ星座なら同じ結果）
	function seeded(seed: string) {
		let h = 2166136261;
		for (const ch of seed) {
			h ^= ch.charCodeAt(0);
			h = Math.imul(h, 16777619);
		}
		return () => {
			h ^= h << 13;
			h ^= h >>> 17;
			h ^= h << 5;
			return ((h >>> 0) % 100000) / 100000;
		};
	}

	const pick = <T,>(rand: () => number, list: T[]) => list[Math.floor(rand() * list.length)];
	const stars = (rand: () => number) => 1 + Math.floor(rand() * 5);

	const ymd = (d: Date) =>
		`${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;

	let reading = $derived.by(() => {
		if (!birthday) return null;
		const [, m, d] = birthday.split('-').map(Number);
		if (!m || !d) return null;
		const sign = signOf(m, d);
		const rand = seeded(`${ymd(today)}:${sign.name}`);
		const scores = { total: stars(rand), love: stars(rand), work: stars(rand), money: stars(rand) };
		return {
			sign,
			scores,
			total: pick(rand, messages.total),
			love: pick(rand, messages.love),
			work: pick(rand, messages.work),
			money: pick(rand, messages.money),
			lucky: {
				color: pick(rand, messages.color),
				item: pick(rand, messages.item),
				number: 1 + Math.floor(rand() * 9)
			},
			rank: 1 + Math.floor(rand() * 12)
		};
	});

	const todayLabel = $derived(
		today.toLocaleDateString('ja-JP', { year: 'numeric', month: 'long', day: 'numeric', weekday: 'short' })
	);
</script>

<svelte:head>
	<title>今日の星占い</title>
	<meta name="description" content="SvelteKit 製の星占いアプリ" />
</svelte:head>

<section class="card">
	<h1>今日の星占い</h1>
	<p class="lead">誕生日を入れると、{todayLabel}の運勢を表示します。</p>

	<label class="field">
		<span>誕生日</span>
		<input type="date" bind:value={birthday} max={ymd(today)} />
	</label>

	{#if reading}
		<div class="result" style="--tone: {elementTone[reading.sign.element]}">
			<p class="symbol" aria-hidden="true">{reading.sign.symbol}</p>
			<p class="sign">{reading.sign.name}</p>
			<p class="sub">{reading.sign.element}の星座　｜　今日の順位 <strong>{reading.rank}位</strong> / 12</p>

			<p class="message">{reading.total}</p>

			<dl class="scores">
				{#each [['総合', reading.scores.total, ''], ['恋愛', reading.scores.love, reading.love], ['仕事', reading.scores.work, reading.work], ['金運', reading.scores.money, reading.money]] as [label, score, text]}
					<div>
						<dt>{label}</dt>
						<dd>
							<span class="stars" aria-label="{score} / 5">
								{'★'.repeat(Number(score))}<span class="off">{'★'.repeat(5 - Number(score))}</span>
							</span>
							{#if text}<span class="text">{text}</span>{/if}
						</dd>
					</div>
				{/each}
			</dl>

			<dl class="lucky">
				<div><dt>ラッキーカラー</dt><dd>{reading.lucky.color}</dd></div>
				<div><dt>ラッキーアイテム</dt><dd>{reading.lucky.item}</dd></div>
				<div><dt>ラッキーナンバー</dt><dd>{reading.lucky.number}</dd></div>
			</dl>
		</div>
	{:else}
		<p class="note">誕生日を選ぶと結果が表示されます。</p>
	{/if}

	<p class="note small">※ 占いの結果は日付と星座から決まる娯楽用のものです。</p>
</section>

<style>
	.card {
		background: var(--surface);
		border: 1px solid var(--line);
		border-radius: 1rem;
		padding: 2rem 1.5rem;
		text-align: center;
	}

	h1 {
		margin: 0 0 0.25rem;
		font-size: 1.4rem;
	}

	.lead {
		margin: 0 0 1.5rem;
		color: var(--muted);
		font-size: 0.9rem;
	}

	.field {
		display: inline-flex;
		align-items: center;
		gap: 0.75rem;
		font-weight: 600;
	}

	input[type='date'] {
		font: inherit;
		padding: 0.6rem 0.9rem;
		border: 1px solid var(--line);
		border-radius: 999px;
		background: var(--surface);
		color: inherit;
	}

	.result {
		margin-top: 2rem;
		padding-top: 1.5rem;
		border-top: 1px dashed var(--line);
	}

	.symbol {
		margin: 0;
		font-size: 3.5rem;
		line-height: 1;
		color: var(--tone);
	}

	.sign {
		margin: 0.25rem 0 0;
		font-size: 2rem;
		font-weight: 800;
		letter-spacing: 0.1em;
		color: var(--tone);
	}

	.sub {
		margin: 0.25rem 0 0;
		font-size: 0.85rem;
		color: var(--muted);
	}

	.sub strong {
		color: var(--tone);
	}

	.message {
		margin: 1.25rem auto 1.5rem;
		max-width: 26rem;
		line-height: 1.7;
	}

	dl {
		display: grid;
		gap: 0.5rem;
		margin: 0 auto;
		text-align: left;
		max-width: 24rem;
	}

	dl > div {
		display: flex;
		justify-content: space-between;
		gap: 1rem;
		border-bottom: 1px solid var(--line);
		padding-bottom: 0.35rem;
	}

	dt {
		color: var(--muted);
		font-size: 0.85rem;
		white-space: nowrap;
	}

	dd {
		margin: 0;
		font-weight: 600;
		text-align: right;
	}

	.scores dd {
		display: flex;
		flex-direction: column;
		align-items: flex-end;
		gap: 0.1rem;
	}

	.stars {
		color: var(--tone);
		letter-spacing: 0.1em;
	}

	.stars .off {
		color: var(--line);
	}

	.text {
		font-weight: 400;
		font-size: 0.8rem;
		color: var(--muted);
	}

	.lucky {
		margin-top: 1.25rem;
	}

	.note {
		margin-top: 1.5rem;
		font-size: 0.85rem;
		color: var(--muted);
	}

	.small {
		font-size: 0.75rem;
	}
</style>
