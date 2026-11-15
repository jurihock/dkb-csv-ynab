<script lang="ts">

  import IconX from './assets/dkb.ico';
  import IconY from './assets/ynab.ico';

  import Drop from './lib/drop.svelte'
  import Table from './lib/table.svelte'

  let data = $state<any>(null);
  let error = $state<string>('');
  let ynab = $state(true);

  function update(dkbcols: string[], dkbrows: Record<string, string>[], name: string) {
    function parseGermanDate(date: string) {
      let [day, month, year] = date.split('.').map(Number);
      if (year < 100) {
        year += 2000;
      }
      return [
        String(year),
        String(month).padStart(2, '0'),
        String(day).padStart(2, '0')
      ].join('-');
    }

    const parseGermanAmount = (amount: string) => {
      if (!amount) {
        return 0;
      }
      return parseFloat(amount
        .replace(/[^\d,-]/g, '') // remove currency symbols or spaces
        .replace(/\./g, '')      // remove thousands separator
        .replace(',', '.')       // replace decimal comma with dot
      );
    };

    // --------------------------------------------------------------
    // DKB columns
    // https://github.com/hamvocke/dkb2homebank/tree/master/testfiles
    // --------------------------------------------------------------
    // Buchungsdatum
    // Wertstellung
    // Status
    // Zahlungspflichtige*r
    // Zahlungsempfänger*in
    // Verwendungszweck
    // Umsatztyp
    // IBAN
    // Betrag (€)
    // Gläubiger-ID
    // Mandatsreferenz
    // Kundenreferenz

    // ----------------------------------------------------------------------
    // YNAB columns
    // https://support.ynab.com/en_us/file-based-import-a-guide-Bkj4Sszyo#csv
    // ----------------------------------------------------------------------
    const ynabcols = ['Date', 'Payee', 'Memo', 'Outflow', 'Inflow'];
    const ynabrows = [];

    error = '';

    try {
      for (let i = 0; i < dkbrows.length; i++) {
        const dkbrow = dkbrows[i];
        const date = parseGermanDate(dkbrow['Wertstellung'] || dkbrow['Buchungsdatum'] || '');
        const payee = dkbrow['Zahlungsempfänger*in'] || dkbrow['IBAN'] || '';
        const memo = dkbrow['Verwendungszweck'] || '';
        const amount = parseGermanAmount(dkbrow['Betrag (€)'] || dkbrow['Betrag'] || '');

        ynabrows.push({
          Date: date,
          Payee: payee,
          Memo: memo,
          Outflow: Math.abs(Math.min(amount, 0)).toString(),
          Inflow:  Math.abs(Math.max(amount, 0)).toString()
        });
      }
    } catch (e) {
      error = String(e);
    }

    data = {
      name: name,
      dkb:  { cols: dkbcols,  rows: dkbrows  },
      ynab: { cols: ynabcols, rows: ynabrows },
    };
  }

  function download() {
    if (!data) {
      return;
    }

    const name: string = data.name;
    const cols: string[] = ynab ? data.ynab.cols : data.dkb.cols;
    const rows: Record<string, string>[] = ynab ? data.ynab.rows : data.dkb.rows;

    const csv = [
      cols.map(col => `"${col}"`).join(','),
      ...rows.map(row => cols.map(col => `"${row[col]}"`).join(','))
    ].join('\n');

    const blob = new Blob([csv], {
      type: 'text/csv;charset=utf-8;'
    });
    const url = URL.createObjectURL(blob);

    const link = document.createElement('a');
    link.href = url;
    link.download = name;

    document.body.appendChild(link);
    link.click();

    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }

</script>

{#if error}

  <div class="empty flex flex-col min-h-screen w-full">
    <p>{error}</p>
  </div>

{:else if data}

  <div class="card border-0 ring-0 shadow-none">
    <header>
      <div class="flex items-center gap-3">
        <div role="group" class="button-group">
          <a class="btn" data-variant="outline" href="{import.meta.env.BASE_URL}">Upload</a>
          <button type="button" class="btn" data-variant="outline" onclick={download}>Download</button>
        </div>
        <div role="group" class="field" data-orientation="horizontal">
          <input id="ynab" type="checkbox" role="switch" class="input" bind:checked={ynab}/>
          <label for="ynab">
             {#if ynab}
              <img class="icon" src={IconY} alt="YNAB"/>
            {:else}
              <img class="icon" src={IconX} alt="DKB"/>
            {/if}
          </label>
        </div>
        <span class="badge ml-auto" data-variant="outline">{data.name}</span>
      </div>
    </header>
    <section>
      {#if ynab}
        <Table cols={data.ynab.cols} rows={data.ynab.rows}/>
      {:else}
        <Table cols={data.dkb.cols} rows={data.dkb.rows}/>
      {/if}
    </section>
  </div>

{:else}

  <Drop onDrop={update} onError={(message) => error = message}/>

{/if}
