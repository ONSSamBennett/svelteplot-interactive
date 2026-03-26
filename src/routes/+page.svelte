<script>
  import { Embed, Radios } from "@onsvisual/svelte-components";
  import BarChart from "../charts/BarChart.svelte"
  import Beeswarm from "../charts/Beeswarm.svelte"

  import rawData from "../charts/shared/data.json"
  import rawData2 from "../charts/shared/data2.json"
  import rawData3 from "../charts/shared/data3.json"

  console.log(rawData)

  let width = $state(700)

  let selectedValue = $state(null)
  let selectedValue2 = $state(null)
  let selectedValue3 = $state(null)

  let data = $derived(selectedValue ? rawData[selectedValue.value] : rawData[0])

  let data2 = $derived(selectedValue2 ? rawData2[selectedValue2.value] : rawData2[0])

  let data3 = $derived(selectedValue3 ? rawData3[selectedValue3.value] : rawData3[0])

  let items = [
    {
      id: "data1",
      label: "Data 1",
      value: 0
    },
    { 
      id: "data2", 
      label: "Data 2",
      value: 1
    },
    { 
      id: "data3", 
      label: "Data 3",
      value: 2
    }
  ];

  let items2 = [
    {
      id: "data2-1",
      label: "Data 1",
      value: 0
    },
    { 
      id: "data-2-2", 
      label: "Data 2",
      value: 1
    }
  ];

  let items3 = [
    {
      id: "data3-1",
      label: "Data 1",
      value: 0
    },
    { 
      id: "data-3-2", 
      label: "Data 2",
      value: 1
    }
  ];

</script>

<Embed>
  <Radios items={items3} bind:value={selectedValue3} label="Select one" compact></Radios>
  <div class="chart-container" bind:clientWidth={width}>
    <BarChart
      variant="simple"
      data={data3}
      xKey="value"
      yKey="series"
      highlighted="Series A"
      dataLabels={true}
      ySort="descending"
      {width}
    />
  </div>

  <Radios {items} bind:value={selectedValue} label="Select one" compact></Radios>
  <div class="chart-container">
    <BarChart
      variant="stacked"
      data={data}
      xKey="value"
      yKey="series"
      zKey="category"
      ySort="descending"
      highlighted="Series A"
      dataLabels={true}
      {width}
    />
  </div>
  <!-- <div class="chart-container">
    <BarChart
      variant="clustered"
      data={data}
      xKey="value"
      yKey="series"
      zKey="category"
      ySort="descending"
      highlighted="Series A"
      dataLabels={true}
      {width}
    />
  </div> -->
  <!-- <Radios items={items2} bind:value={selectedValue2} label="Select one" compact></Radios>
    <div class="chart-container">
    <Beeswarm
      variant="binned"
      data={data2}
      xFormat={".0%"}
      highlighted="Stroud"
    />
    </div> -->
</Embed>