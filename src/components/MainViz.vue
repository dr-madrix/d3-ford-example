<template lang="pug">
.container.text-center.py-2
    button.btn.btn-primary(@click="firstViz") Show Data 
#vizdiv.container.mt-4.py-2
    .row
        #years.col-6.border.border-start-0.border-end-0.border-dark 
            p.text-center.mt-2.text-success.fw-bolder
            svg(ref="daSvg")
        #quarters.col-6.border.border-dark 
            p.text-center.mt-2.text-success.fw-bolder
            svg
</template>

<script>
import fordData from "../assets/prj_ford"
import * as d3 from 'd3';

let years = new Set();
fordData.forEach(d => {
    years.add(d.year)
});
years = Array.from(years);

let yearlyData = [];
years.forEach(year => {
    let yearSum = 0;
    fordData.forEach(d => {
        if(d.year == year){
            yearSum += d.truck + d.suv + d.car;
        }
    })
    yearlyData.push(yearSum);
})

console.log(fordData)

export default {
    data(){
        return{
            fordData: fordData,
            yearlyData: yearlyData,
            years: years
        }
    },
    methods: {
        firstViz(){
            d3.select("button")
                .remove()

            let fordData = this.fordData;
            d3.select("#years p")
                .text("Number of Vehicles")
                .append("p")
                .text("(click for quarterly break up)")
                .classed("text-muted", true)
                .style("font-size", "smaller")

            let dimensions = {
                height: this.$refs.daSvg.clientHeight,
                width: this.$refs.daSvg.clientWidth
            }
            
            const xScale = d3.scaleLinear()
                .domain([0, Math.max(...yearlyData)])
                .range([0, dimensions.width - 130])
            
            const yScale = d3.scaleLinear()
                .domain([0, yearlyData.length - 1])
                .range([30, dimensions.height - 150])

            const yearSvg = d3.select("#years svg")
                .selectAll("rect")
            
            yearSvg.data(this.yearlyData)
                .join("rect")
                .attr("x", "0")
                .attr("y", function(d, i){
                    return yScale(i);
                })
                .attr("height", function(d, i){
                    return dimensions.height / 4 - 5;
                })
                .attr("width", d => xScale(d))
                .style("fill", (d, i)=>{
                    if(i === 0){
                        return "steelblue"
                    } else {
                        return "dodgerblue"
                    }
                })
                .style("cursor", "pointer")
                .attr("id", (d, i) => `${this.years[i]}`)

            yearSvg.data(this.yearlyData)
                .join("text")
                .attr("x", d => xScale(d) + 10)
                .attr("y", (d, i) => {
                    return yScale(i) + 35;
                })
                .text((d, i) => `${this.years[i]}: ${d}`)

            d3.select("#quarters p")
                .text("Click on a year for more details")
            
            d3.select("#years")
                .selectAll("rect")
                .on("click", function(e, d){
                    d3.select("#quarters p")
                        .text(`${e.target.id}: Quarterly Break up`)

                    let quarterlyData = [];
                  
                    fordData.forEach((d)=>{
                        if(e.target.id == d.year){
                            quarterlyData.push(d)
                        }
                    })

                    quarterlyData = quarterlyData.map(obj => {
                        obj.qTotal = Number(obj.suv) + Number(obj.truck) + Number(obj.car)
                        let {suv, truck, car, ...rest} = obj;
                        return rest
                    })

                    let qSales = quarterlyData.map(obj => obj.qTotal)

                    console.log(qSales)

                    const xScale2 = d3.scaleLinear()
                                    .domain([0, Math.max(...qSales)])
                                    .range([0, dimensions.width - 130])

                    const yScale2 = d3.scaleLinear()
                        .domain([0, yearlyData.length - 1])
                        .range([30, dimensions.height - 170])

                    const quarterlySvg = d3.select("#quarters svg")

                        quarterlySvg.selectAll("rect")
                            .data(quarterlyData)
                            .join("rect")
                            .attr("x", "0")
                            .attr("y", function(d, i){
                                return yScale2(i) ;
                            })
                            .attr("height", function(d, i){
                                return dimensions.height / 6 - 5
                            })
                            .attr("width", d => xScale2(d.qTotal))
                            .attr("id", d => d.quarter)
                            .style("fill", "skyblue")
                            .style("cursor", "pointer")

                        quarterlySvg.selectAll("text")
                            .data(quarterlyData)
                            .join("text")
                            .attr("x", d => xScale2(d.qTotal) + 5)
                            .attr("y", (d, i) => {
                                return yScale2(i) + 20; 
                            })
                            .text((d,i) => `${d.quarter}: ${d.qTotal}`)

                    
                })
        }
    }

}
</script>

<style scoped>
#vizdiv{
    min-height: 300px;
}

#vizdiv .row{
    min-height: 300px;
}

.row div svg{
    height: 80%;
    width: 100%;
}
</style>