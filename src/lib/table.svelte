<script lang="ts">

  interface Props {
    cols: string[];
    rows: Record<string, string>[];
  }

  let { cols, rows }: Props = $props();

</script>

<div class="table-container">
  <table class="table">
    <thead>
      <tr>
        {#each cols as col}
          <th>{col}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each rows as row}
        <tr>
          {#each cols as col}
            <td>
              {row[col]}
            </td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
</div>