<script setup>
import { ref, shallowRef, provide, computed } from "vue";
import VChart, { THEME_KEY } from "vue-echarts";

import { use } from "echarts/core";
import { TooltipComponent, VisualMapComponent } from 'echarts/components'
import { CanvasRenderer } from 'echarts/renderers'
import { SurfaceChart, Scatter3DChart } from 'echarts-gl/charts'
import { Grid3DComponent } from 'echarts-gl/components'

use([
    TooltipComponent,
    VisualMapComponent,
    Grid3DComponent,
    SurfaceChart,
    Scatter3DChart,
    CanvasRenderer
])

provide(THEME_KEY, "light");

const x0 = ref(0)
const y0 = ref(0)
const funcion = shallowRef({label:'Paraboloide hiperbólico', valor: (x,y) => x**2 - y**2})
const items = [
    {label:'Paraboloide hiperbólico', valor: (x,y) => x**2 - y**2},
    {label:'Gaussiana modificada', valor: (x,y) =>
        x**2 + y**2 <0.2**2 ? 0.2**2
        : 1/Math.exp(4*(x**2+y**2)) -0.02/(x**2 + y**2)},
    {label:'Trigonométrica', valor: (x,y) => (Math.sin(4*x)+Math.cos(4*y))/3}
]

const h = 1e-4
const der = {
    fx:   (f, x, y) => (f(x+h, y)   - f(x-h, y))   / (2*h),
    fy:   (f, x, y) => (f(x, y+h)   - f(x, y-h))   / (2*h),
    fxx:  (f, x, y) => (f(x+h, y)   - 2*f(x,y) + f(x-h, y))   / (h*h),
    fyy:  (f, x, y) => (f(x, y+h)   - 2*f(x,y) + f(x, y-h))   / (h*h),
    fxy:  (f, x, y) => (f(x+h,y+h)  - f(x+h,y-h) - f(x-h,y+h) + f(x-h,y-h)) / (4*h*h),
}

const order = shallowRef(1)
const taylor = (f,n) =>
    (x,y) => [
        f(x0.value, y0.value),
        (x - x0.value) * der.fx(f, x0.value, y0.value)
        + (y - y0.value) * der.fy(f, x0.value, y0.value),
        1/2 * (x - x0.value)*(x - x0.value) * der.fxx(f, x0.value, y0.value)
        + 1/2 * (x - x0.value)*(y - y0.value) * der.fxy(f, x0.value, y0.value)
        + 1/2 * (y - y0.value)*(x - x0.value) * der.fxy(f, x0.value, y0.value)
        + 1/2 * (y - y0.value)*(y - y0.value) * der.fyy(f, x0.value, y0.value)
    ].slice(0,n+1).reduce((acc, val) => acc + val, 0)

const option = computed(() => ({
        tooltip: {},
        backgroundColor: '#fff',
        visualMap: {
            show: false,
            dimension: 2,
            min: -1,
            max: 1,
            seriesIndex: 0,
            inRange: {
                color: [
                    '#313695',
                    '#4575b4',
                    '#74add1',
                    '#abd9e9',
                    '#e0f3f8',
                    '#ffffbf',
                    '#fee090',
                    '#fdae61',
                    '#f46d43',
                    '#d73027',
                    '#a50026'
                ]
            }
        },
        xAxis3D: {
            type: 'value'
        },
        yAxis3D: {
            type: 'value'
        },
        zAxis3D: {
            type: 'value',
            min: -1,
            max: 1
    },
        animation: false,
        grid3D: {
            viewControl: {
                // projection: 'orthographic'
            }
        },
        series: [
            {
                type: 'surface',
                itemStyle: {
                    opacity:0.8
                },
                equation: {
                    x: {
                        step: 0.05
                    },
                    y: {
                        step: 0.05
                    },
                    z: funcion.value.valor
                }
        },{
            type: 'scatter3D',
            itemStyle : {
                color: '#000000',
                opacity: 1
            },
            data: [
                {
                    name: 'center',
                    value: [
                        x0.value,
                        y0.value,
                        funcion.value.valor(x0.value, y0.value)],
                    color: '#e74c3c'
                }
            ]
        },{
            type: 'surface',
            itemStyle: {
                opacity:1
            },
            equation: {
                x: {
                    step: 0.05
                },
                y: {
                    step: 0.05
                },
                z: taylor(funcion.value.valor, order.value)
            }
        }
    ]
}))
</script>

<template>
    <v-app theme="light">
        <v-main>
            <v-container>
                <h2 class="text-center">Serie de Taylor multivariable</h2>
                <v-card>
                    <v-row>
                        <v-col cols="8">
                            <v-select
                                class="py-6 pl-6"
                                v-model="funcion"
                                :items="items"
                                item-title="label"
                                item-value="valor"
                                return-object
                                label="Función"
                                variant="outlined"
                            ></v-select>
                        </v-col>
                        <v-col cols="4">
                            <v-select
                                class="py-6 pr-6"
                                v-model="order"
                                :items="[0,1,2]"
                                label="Orden"
                                variant="outlined"
                            ></v-select>
                        </v-col>
                    </v-row>
                    <v-chart
                        :option="option"
                        autoresize
                        style="height: 50dvh"
                    />
                    <v-slider
                        class="px-6"
                        v-model="x0"
                        min="-1"
                        max="1"
                        label="x0"
                    ></v-slider>
                    <v-slider
                        class="px-6"
                        v-model="y0"
                        min="-1"
                        max="1"
                        label="y0"
                    ></v-slider>
                </v-card>
            </v-container>
        </v-main>
    </v-app>
</template>

<style scoped></style>
