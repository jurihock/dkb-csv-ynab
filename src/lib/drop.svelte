<script lang="ts">

  import Papa from 'papaparse';

  interface Props {
    onDrop: (cols: string[], rows: Record<string, string>[], filename: string) => void;
    onError?: (message: string) => void;
  }

  let { onDrop, onError }: Props = $props();

  let isDragging = $state(false);
  let fileInput: HTMLInputElement;

  function handleDrop(event: DragEvent) {
    event.preventDefault();
    isDragging = false;
    const file = event.dataTransfer?.files?.[0];
    processFile(file);
  }

  function handleFileInput(event: Event) {
    const target = event.target as HTMLInputElement;
    processFile(target.files?.[0]);
    target.value = ''; // reset input so the same file can be re-uploaded
  }

  async function processFile(file: File | undefined) {
    if (!file || !file.name.toLowerCase().endsWith('.csv') && file.type !== 'text/csv') {
      onError?.('Please upload a valid CSV file.');
      return;
    }

    try {
      const text = await file.text();
      const lines = text.split(/\r?\n/);
      const offset = lines.findIndex(line =>
        line.startsWith('"Buchungsdatum"') ||
        line.startsWith('Buchungsdatum'));
      const csv = lines.slice(offset).join('\n');

      const results = Papa.parse(csv, {
        header: true,
        skipEmptyLines: true,
        dynamicTyping: false,
        delimiter: ';'
      });

      if (results.errors.length > 0) {
        onError?.(results.errors[0].message);
      } else {
        const cols = results.meta.fields as string[];
        const rows = results.data as Record<string, string>[];
        if (cols.length > 0 && rows.length > 0) {
          onDrop(cols, rows, file.name);
        }
      }
    } catch (error) {
      onError?.(String(error));
    }
  }

</script>

<div
  role="button"
  tabindex="0"
  class="flex flex-col min-h-screen w-full"
  class:dragging={isDragging}
  ondragover={(e) => { e.preventDefault(); isDragging = true; }}
  ondragleave={() => (isDragging = false)}
  ondrop={handleDrop}
  onclick={() => fileInput.click()}
  onkeydown={(e) => e.key === 'Enter' && fileInput.click()}
>
  <input
    bind:this={fileInput}
    type="file"
    accept=".csv,text/csv"
    onchange={handleFileInput}
    hidden
  />
  <p class="empty">{isDragging
    ? 'Drop your DKB CSV file here...'
    : 'Drag & drop a DKB CSV file here or click to browse...'}
  </p>
</div>
