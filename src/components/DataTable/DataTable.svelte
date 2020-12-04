<script>
  import path from 'path-browserify';
  import { createEventDispatcher } from "svelte";
  import { slide } from "svelte/transition";
  import { ClassBuilder } from "../../utils/classes.js";
  import Icon from "../Icon";
  import Button from "../Button";
  import TextField from "../TextField";
  import ProgressLinear from "../ProgressLinear";

  import Header from "./Header.svelte";
  import Row from "./Row.svelte";
  import Pagination from "./Pagination.svelte";

  import defaultSort from "./sort.js";

  const classesDefault = "elevation-2 relative text-sm overflow-x-auto w-full dark:bg-dark-500";

  export let data = [];
  export let datafile = "";
  export let columns = Object.keys(data[0] || {})
    .map(i => ({ label: (i || "").replace("_", " "), field: i }));

  export let page = 1;
  export let sort = defaultSort;
  export let perPage = 10;
  export let perPageOptions = [10, 20, 50];
  export let asc = false;
  export let loading = false;
  export let hideProgress = false;
  export let editable = false;
  export let sortable = true;
  export let pagination = true;
  export let scrollToTop = false;
  export let headerClasses = i => i;
  export let paginationClasses = i => i;
  export let editableClasses = i => i;
  export let paginatorProps = null;
  export let classes = classesDefault;

  let table = "";
  let sortBy = null;

  $: {
    perPage = pagination ? perPage : data.length;
  }
  $: offset = (page * perPage) - perPage;
  $: sorted = sort(data, sortBy, asc).slice(offset, perPage + offset);
  $: pagesCount = Math.ceil(data.length / perPage);

  const dispatch = createEventDispatcher();

  let editing = false;

  const cb = new ClassBuilder(classes, classesDefault);
    $: c = cb
      .flush()
      .add(classes, true, classesDefault)
      .add($$props.class)
      .get();

  const download = (href, filename) => {
    const alink = document.createElement("a");
    alink.href = href;
    alink.download = filename;
    document.body.appendChild(alink);
    alink.click();
    document.body.removeChild(alink);
  };

  const downloadTable = () => {
    const href = 'data:application/vnd.ms-excel;charset=utf-8,\ufeff'+ encodeURIComponent(function(){
      const dataTitle = columns.map(col => col.label);
      const dataMain = [];

      data.forEach(row => {
        const vals = columns.map(col => `"${row[col.field]}"`);

        dataMain.push(vals.join(','));
      });

      return dataTitle.join(',') + '\r\n' + dataMain.join('\r\n');
    }());

    const filename = datafile.length > 0
      ? path.parse(datafile).name + '.xls'
      : 'data-subset.xls';

    download(href, filename);
  }

</script>

<table class={c} bind:this={table}>
  <thead class="items-center">
    {#each columns as column, i}
      <slot name="header">
        <Header
          class={headerClasses}
          {column}
          bind:asc
          bind:sortBy
          {sortable}
          {editing}
        />
      </slot>
    {/each}
  </thead>
  {#if loading && !hideProgress}
    <div class="absolute w-full" transition:slide>
      <ProgressLinear />
    </div>
  {/if}
  <tbody>
    {#each sorted as item, index}
      <slot name="item">
        <Row
          bind:editing
          {index}
          {item}
          {columns}
          {editable}
          {editableClasses}
          on:update
        />
      </slot>
    {/each}
  </tbody>
  {#if pagination}
    <slot name="pagination">
      <Pagination
        bind:page
        bind:perPage
        on:downloadTable={downloadTable}
        class={paginationClasses}
        {datafile}
        {perPageOptions}
        {scrollToTop}
        {paginatorProps}
        {offset}
        {pagesCount}
        {table}
        total={data.length}
      />
    </slot>
  {/if}

  <slot name="footer" />
</table>
