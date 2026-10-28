<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  // Counted the desk's way: every report where it is filed. The one-day move (desk-notes.json)
  // is its own problem; its reports are also counted under the cause of their tenant.
  const problem = (r: Report) => causeOf(r, tenantOf(r)) ?? 'unplaced';
  const count = (p: string) => merged.filter((r) => problem(r) === p);

  const rows = $derived([
    ...['parser', 'hint', 'unplaced'].map((p) => ({ problem: p, rs: count(p) })),
  ].map(({ problem, rs }) => ({ problem, count: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size })));
</script>

<p><strong>{merged.length}</strong> unique reports merged into #{ticket}.</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th></tr></thead>
  <tbody>
    {#each rows.filter((r) => r.count) as r (r.problem)}
      <tr><td><code>{r.problem}</code></td><td>{r.count}</td><td>{r.tenants}</td></tr>
    {/each}
  </tbody>
</table>
