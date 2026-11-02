<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { PROBLEM_TITLES, splitTicket, type Problem } from '#lib/kb/problems';

  // issues: problem → issue number in `repo` (GitHub "owner/name").
  let { ticket, repo, issues = {} }: { ticket: number; repo: string; issues?: Partial<Record<Problem, number>> } = $props();

  const split = $derived(splitTicket(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const channels = (c: Record<string, number>) =>
    Object.entries(c)
      .sort((a, b) => b[1] - a[1])
      .map(([ch, n]) => `${ch} ×${n}`)
      .join(' · ');
</script>

<p class="sum">
  <strong>{split.merged.length}</strong> reports merged into #{ticket} ·
  <strong>{split.groups.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>channels</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code><br />{PROBLEM_TITLES[g.problem]}</td>
        <td><strong>{g.reports.length}</strong></td>
        <td>{g.tenants.length}</td>
        <td class="muted">{channels(g.channels)}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if issues[g.problem]}
            <a href="https://github.com/{repo}/issues/{issues[g.problem]}">{repo}#{issues[g.problem]}</a>
          {:else}
            <span class="warn">none yet</span>
          {/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

{#each split.groups as g (g.problem)}
  <details>
    <summary><code>{g.problem}</code>: what the reports say ({Math.min(g.reports.length, 5)} of {g.reports.length})</summary>
    <ul>
      {#each g.reports.slice(0, 5) as r (r.id)}
        <li><code>{r.id}</code> · {r.tenant} · {r.channel}: “{r.body}”</li>
      {/each}
    </ul>
  </details>
{/each}

<style>
  details { margin: 0.4rem 0; font-size: 0.9rem; }
  summary { cursor: pointer; }
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .warn { color: var(--warn); font-weight: 600; }
</style>
