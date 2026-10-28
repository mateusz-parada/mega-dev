<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  // The one-day move is its own problem (desk-notes.json); the rest split by cause.
  const problem = (r: Report) => (notes.oneDayMove.includes(r.id) ? 'one-day-move' : (causeOf(r, tenantOf(r)) ?? 'unplaced'));

  const rows = $derived(
    ['parser', 'hint', 'one-day-move', 'unplaced'].map((p) => {
      const rs = merged.filter((r) => problem(r) === p);
      return { problem: p, count: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size };
    }),
  );
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
