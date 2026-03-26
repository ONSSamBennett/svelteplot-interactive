<script>

    import { Plot, Dot, Text, RectX, Pointer, HTMLTooltip } from 'svelteplot';
    import { format } from "d3-format";
    import { timeParse, timeFormat} from "d3-time-format"
    import * as d3 from 'd3';
    import { onMount, untrack } from 'svelte';
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
        legendItemLabel = "Local authorities",
        smGridPosition,
        smKey,
        xKey = "x", 
        yKey = Object.keys(data[0]).includes('y') ? 'y' : null,
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
        tooltip = true,
        height,
        seriesHeight = 200,
        radius = 4,
        padding = -3.5,
        numberBins = 75,
        hover = false,
        margin = {top: 0, bottom: 0, right: 20, left: 20}, 
        colours = defaultColours,
        children
    } = $props();

    let plotEl = $state();
    let tooltipData = $state();
    let leaveTimeout;

    $inspect(data)

    let categories = $derived(yKey ? new Set(data.map((d) => d[yKey])) : null)

    let colourLookup = $derived.by(() => {
        let coloursvar;
        coloursvar = {}
        let i = 0
        if(colours.length == 1){
            coloursvar[legendItemLabel] = colours
        }
        else{
            categories.forEach((category) => {
                coloursvar[category] = colours[i]
                i = i+1
            })
        }
        if(highlighted){
            coloursvar[highlighted] = ONScolours.highlightOrange
        }
        return coloursvar
    })

    let legendCategories = $derived.by(() => {
        const coloursLength = Object.keys(colourLookup).length
        if(coloursLength == 1){
            return null
        } else if(colours.length == 1 && highlighted){
            return [legendItemLabel, highlighted]
        } else{
            return [...categories, highlighted]
        }
    })


    let domainX = $derived(getContinuousDomain({
        data: data,
        variant: variant,
        categoryKey: yKey,
        valueKey: xKey,
        xDomain: xDomain
    }))

    let chartHeight = $derived(height ? height : getChartHeight({data: data, seriesHeight: seriesHeight, categoryKey: yKey, variant: variant}))

    let domainY = $derived(!yKey ? null : yDomain)

    let bins = $derived.by(() => {
        if(variant == 'force'){
            return null
        } else{
            let binSize = Math.abs(domainX[1] - domainX[0])/numberBins
            console.log(binSize)
            let binsArr = []
            for(let i = 0; i < numberBins; i++){
                binsArr.push(domainX[0] + (i * binSize))
            }
            return binsArr;
        }
    })

    let dodgeY = $derived.by(() => {
        if(variant == 'binned' || variant == 'binned-diverging'){
            return 'bottom'
        } else{
            return 'middle'
        }
    })

    let binSize = $derived(bins ? Math.abs(domainX[1] - domainX[0]) / numberBins : null)

    let dots = $state([])

    $effect(() => {
        data.forEach((d) => {
            let binX = null
            if (bins && binSize) {
                const binIndex = Math.min(
                    Math.floor((d[xKey] - domainX[0]) / binSize),
                    numberBins - 1
                )
                binX = bins[binIndex] + binSize / 2
            }
            let obj = {...d, code: d[zKey].replace(/[^A-Z0-9\s]+|\s+/ig, "") + d[yKey].replace(/[^A-Z0-9\s]+|\s+/ig, ""), binX}
            untrack(() => {
                if(keyIndex.indexOf(obj[zKey]) != -1){
                    dots[keyIndex.indexOf(obj[zKey])] = obj
                } else{
                    dots.push(obj)
                }
            })
        })
    })

    let keyIndex = $derived(dots ? dots.map((d) => d[zKey]) : [null])

    $effect(() => {
        if (dots && plotEl){
            d3.select(plotEl).selectAll('.axis-y').attr('display','none')
        }
    })

    $effect(() => {
        if(highlighted){
            setTimeout(() => {
                d3.select(plotEl).selectAll(".highlighted").raise()
            }, 100);
        }
    })

</script>

{#if legendCategories}
    <Legend categories={legendCategories} colourScheme={colourLookup}/>
{/if}


<div bind:this={plotEl}>

<Plot 
    marginLeft={margin.left}
    marginRight={margin.right ? margin.right : null}
    marginTop={margin.top ? margin.top : null}
    marginBottom={margin.bottom ? margin.bottom : null}
    height = {chartHeight} 
    {width}
    y={{
        domain: null,
    }}
    fy={{ 
        axis: null,
        domain: domainY ? domainY : null, 
        tickSpacing: 10, 
        ticks: []
        // tickFormat: () => ""
    }} 
    x={{ 
        domain: domainX, 
        label:xAxisLabel ? xAxisLabel : "",
        tickFormat: (d) => xFormatDate ? timeFormat(xFormat)(timeParse(xFormatDate)(d)) : xFormat ? format(xFormat)(d) : d,
        grid: true
    }}
    color={{ 
        scheme: colours
    }}
>
    {#each categories as category}
        <RectX
            y1={0}
            y2={seriesHeight}
            fy={category}
            insetTop={-2}
            insetBottom={-2}
            insetLeft={-margin.left}
            insetRight={-margin.right}
            fill={ONScolours.grey20}
            opacity={0.25}
        />
        <Text
            x={domainX[0]}
            dy={-(seriesHeight/2) + 25}
            fy={category}
            textAnchor='start'
            text={category}
        />
    {/each}
    {#if dots}
    <Dot
        data={dots}
        x={variant == 'force' ? xKey : 'binX'}
        y={0}
        dotClass={(d) => d[zKey] == highlighted ? d.code + ' highlighted dot' : d.code + ' dot'}
        r={radius}
        dodgeY={{ anchor: dodgeY, padding: padding}}
        fill={(d) => {
            let colour;
            if(colours.length > 1){
                colour = colourLookup[d[yKey]]
            } else{
                colour = colours
            }
            return colour
        }} 
        stroke={(d) => {
            if(d[zKey] == highlighted){
                return ONScolours.highlightOrange
             } else{
                return ONScolours.white
             }
        }}
        strokeWidth={(d) => {
            if(highlighted && d[zKey] == highlighted){
                return 8
            } else{
                return 1
            }
        }}
        onpointerenter={(evt, d) => {
            if(tooltip) {
                clearTimeout(leaveTimeout);
                tooltipData = {x: evt.layerX, y: evt.layerY, data: d}

                d3.select(plotEl).selectAll(".point-highlight").remove();
                  // Select all matching elements and clone them as highlights
                d3.select(plotEl).selectAll('.'+d.code)
                    .clone(true)
                    .attr("class", "point-highlight")
                    .attr("fill", "none")
                    .style("pointer-events", "none")
                    .style("stroke", ONScolours.grey100)
                    .style("stroke-width", 4)
                    .raise();
            }
        }}
        onpointerleave={() => {
            if(tooltip){
                leaveTimeout = setTimeout(() => {
                    d3.select(plotEl).selectAll(".point-highlight").remove();
                    tooltipData = null
                }, 300)
            }
        }}
    />
    {/if}


    {#snippet overlay()}
        {#if tooltipData}
            {@const isPastMidpoint = tooltipData.data[xKey] > (domainX[0] + domainX[1]) / 2}
            <div 
                class="tooltip"
                style:left="{tooltipData.x}px"
                style:top="{tooltipData.y}px"
                style:transform={isPastMidpoint ? 'translate(-100%, -50%)' : 'translate(10px, -50%)'}
                style:width="max-content"
            >
                <div class="identifier">{tooltipData.data[zKey]}</div>
                <div class="group">{tooltipData.data[yKey]}</div>
                <div class="value">{xAxisLabel ? xAxisLabel+": "+d3.format(xFormat)(tooltipData.data[xKey]) : d3.format(xFormat)(tooltipData.data[xKey])}</div>
            </div>
        {/if}
    {/snippet}

    {#if children}
        {@render children()}
    {/if}
</Plot>
</div>

<style>
    :global(.dot path){
        paint-order: stroke fill;
    }
    .tooltip {
        position: absolute;
        pointer-events: none;
        background: white;
        /* border: 1px solid #ccc; */
        padding: 4px 8px;
        font-size: 14px;
        transform: translate(10px, -50%);
    }
    .tooltip .identifier{
        font-weight: 600;
    }
    .tooltip .value{
        line-height: 14px;
        margin-bottom: 6px;
    }
    .highlighted{
        outline: 'white';
        outline-width: "12px";
    }
    :global(.dot) {
		transition: all 1s ease-in-out;
		opacity: 1;
		animation-name: fadeInOpacity;
		animation-timing-function: ease-in;
		animation-duration: 0.5s;
	}
	@keyframes fadeInOpacity {
		0% {
			opacity: 0;
		}
		100% {
			opacity: 1;
		}
	}
</style>