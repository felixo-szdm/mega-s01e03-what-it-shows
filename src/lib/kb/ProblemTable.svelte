<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  type Problem = 'parser' | 'hint' | 'one-day' | 'unplaced';

  let { ticket, issues }: { ticket: number; issues: Partial<Record<Problem, string>> } = $props();

  // Counted the desk's way (CauseTable): a one-day move stays filed under its tenant's cause,
  // and gets a row of its own because it needs its own fix.
  const exported = (reports as Report[]).filter((r) => r.mergedInto === ticket);
  const merged = $derived(uniqueById(exported));
  const causes = $derived(groupByCause(merged, tenantOf));
  const oneDay = $derived(causes.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0));
  const filedUnder = $derived(causes.filter((g) => readAs(g, notes.oneDayMove) > 0).map((g) => g.cause));
  const rows = $derived([
    ...causes.map((g) => ({ problem: g.cause as Problem, count: g.reports.length, tenants: g.tenants })),
    { problem: 'one-day' as Problem, count: oneDay, tenants: null },
  ]);
  const issueNumber = (url: string) => url.replace(/^.*\/issues\//, '#');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} · <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code></td>
        <td>{row.count}{#if row.problem === 'one-day'}*{/if}</td>
        <td>{row.tenants ?? '—'}</td>
        <td>
          {#if issues[row.problem]}<a href={issues[row.problem]}>{issueNumber(issues[row.problem] ?? '')}</a>{:else}none yet{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  * also counted under <code>{filedUnder.join(', ')}</code>, where the desk files them by tenant. The export has
  {exported.length} rows for #{ticket}; {exported.length - merged.length} are repeats from overlapping export pages and
  count once.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
