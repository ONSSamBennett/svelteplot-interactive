<script>
    import { Plot, Dot, AxisY } from 'svelteplot';
    import { format } from "d3-format";
    import { timeParse, timeFormat} from "d3-time-format"
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { 
        getCategoricalDomain, 
        getContinuousDomain, 
        groupData, 
        stackData, 
        labelPixelWidth, 
        getChartHeight, 
        getSeriesHeight,
        getAxisMargin 
    } from '../js/utils';
    import { ONScolours, ONSpalette, oldONSpalette } from '../js/colours'
    import Legend from "./shared/Legend.svelte"

    let defaultColours = [ONScolours.oceanBlue]

	let { 
        data, 
        size,
        width,
        variant = "force",
        highlighted = null,
        smGridPosition,
        smKey,
        xKey = "x", 
        yKey = "y",
        zKey = "z",
        xAxisLabel,
        yAxisLabel, 
        xDomain = "auto",
        xFormat,
        xFormatDate,
        xAxisTicks,
        yDomain,
        yFormat,
        yFormatDate,
        zFormat,
        zFormatDate,
		ySort,
        zSortKey, 
        dataLabels,
        tooltip,
        height,
        seriesHeight = 200,
        radius = 3,
        hover = false,
        margin = {top: 0, bottom: 0, right: 20, left: 20}, 
        colours = defaultColours,
        children
    } = $props();

    let hovered = $state();

    let categories = $derived(yKey ? new Set(data.map((d) => d[yKey])) : null)

    let colourScheme = $derived.by(() => {
        let coloursvar;
        if(categories){
            coloursvar = {}
            let i = 0
            categories.forEach((category) => {
                coloursvar[category] = colours[i]
                i = i+1
            })
        } else if(colours.length > 1){
            coloursvar = colours[0]
        } else{
            coloursvar = colours
        }
        return coloursvar
    })

    let domainX = $derived(getContinuousDomain({
        data: data,
        variant: variant,
        categoryKey: yKey,
        valueKey: xKey,
        xDomain: xDomain
    }))

    let chartHeight = $derived(height ? height : getChartHeight({data: data, seriesHeight: seriesHeight, categoryKey: yKey, groupKey: zKey, variant: variant}))

    let barHeight = $derived(seriesHeight ? seriesHeight : getSeriesHeight({data: data, height: height, categoryKey: yKey, groupKey: zKey, variant: variant}))

    let domainY = $derived(!yKey ? null : yDomain ? yDomain : getCategoricalDomain({
        data: data, 
        variant: variant, 
        sort: ySort, 
        sortKey: zSortKey, 
        valueKey: xKey, 
        categoryKey: yKey, 
        groupKey: zKey
    }))

    $inspect(domainY)

    let yAxisMargin = $derived(margin.left ? margin.left : getAxisMargin({domain: domainY}))

    let dots = $derived.by(() => {
        let derivedData = []
        data.forEach((d) => {
            derivedData.push({...d, r: d[zKey] == highlighted ? 1 : 0})
        })
        return derivedData
    })

    // onMount(() => {
    //     d3.selectAll(".is-left").attr("text-anchor","start")
    // })

</script>

{#if categories && !smKey && colours.length > 1}
    <Legend {categories} {colourScheme}/>
{/if}

<Plot 
    marginLeft={yAxisMargin}
    marginRight={margin.right ? margin.right : null}
    marginTop={margin.top ? margin.top : null}
    marginBottom={margin.bottom ? margin.bottom : null}
    height = {chartHeight} 
    {width}
    y={{
        domain: null,
    }}
    fy={{ 
        axis: 'left',
        domain: domainY ? domainY : null, 
        tickSpacing: 10, 
        label: yAxisLabel ? yAxisLabel : "",
    }} 
    x={{ 
        domain: domainX, 
        label:xAxisLabel ? xAxisLabel : "",
        tickFormat: (d) => xFormatDate ? timeFormat(xFormat)(timeParse(xFormatDate)(d)) : xFormat ? format(xFormat)(d) : d,
        grid: true
    }}
    r={{
        range: highlighted ? [radius, radius+3] : [radius, radius],
        domain: [0,1]
    }}
    color={{ 
        // legend: variant == "clustered" || variant == "stacked" ? true : false,
        scheme: colours
    }}
>
    <Dot
        data={dots}
        x={xKey}
        y={0}
        fy={yKey}
        r="r"
        dodgeY={{ anchor: variant == 'binned' ? 'bottom' : 'middle', padding: 0 }}
        fill={(d) => {
            let colour;
            if(categories.length > 1){
                colour = colourScheme[d[yKey]]
            } else if(d[zKey] == highlighted){
                colour = ONScolours.highlightOrange
            } else if(highlighted){
                colour = colours[0]
            } else{ 
                colour = colours[0]
            }
            return colour
        }} 
        stroke={(d) => {
            if(highlighted && d[zKey] == highlighted){
                return ONScolours.grey100
            } else{
                return ONScolours.white
            }
        }}
        strokeWidth={(d) => {
            if(highlighted && d[zKey] == highlighted){
                return 2
            } else{
                return 1
            }
        }}
        />

    {#if children}
        {@render children()}
    {/if}
</Plot>

<style>
    :global(.dot path){
        paint-order: stroke fill;
    }
</style>