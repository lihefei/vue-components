<template>
    <canvas
        class="canvas"
        ref="chart"
        :width="width"
        :height="height"
        :style="{
            'background-size': `${width - padding.left - padding.right}px ${height - padding.top - padding.bottom}px`,
            'background-position': `${padding.left}px ${padding.top}px`,
        }"
    ></canvas>
</template>

<script lang="ts">
import { Component, Prop, Watch, Vue } from 'vue-property-decorator';

@Component({ name: 'StationMap' })
export default class extends Vue {
    @Prop({ default: 1200 }) width!: number;
    @Prop({ default: 500 }) height!: number;
    @Prop({
        default() {
            return { left: 0, right: 70, top: 30, bottom: 20 };
        },
    })
    padding!: any; //画布内边距

    //配置
    private option: any = {
        grid: {
            width: 0,
            height: 0,
            left: 0,
            right: 0,
            top: 0,
            bottom: 0,
            center: {
                x: 0,
                y: 0,
            },
        },
        lineWidth: 1,
        fontSize: 12,
        fillStyle: 'rgba(255,255,255,0.5)',
        center: {
            x: 0,
            y: 0,
        },
        ratio: 1,
    };

    private canvas: any = null;
    private ctx: any = null;

    private timer: any = null; //定时器

    private mounted() {
        this.init();
        this.draw();
        this.openTimer();
    }

    private beforeDestroy() {
        this.closeTimer();
    }

    //打开定时器
    private openTimer() {
        this.closeTimer();
        this.timer = setInterval(() => {
            this.clearCanvas();
            this.draw();
        }, 2000);
    }

    //关闭定时器
    private closeTimer() {
        clearInterval(this.timer);
    }

    //绘制
    private async draw() {
        //await this.drawBgMap();
        //this.drawCenterLine();
        //this.drawGridLine();
        this.drawColorColumn();
        this.drawDot();
    }

    //初始化画布
    private init() {
        this.option.center.x = this.width / 2;
        this.option.center.y = this.height / 2;

        this.canvas = this.$refs.chart;
        this.ctx = (this.canvas as HTMLCanvasElement).getContext('2d');
        this.ctx.translate(this.option.center.x, this.option.center.y); //定义中心点

        this.option.grid.width = this.width - (this.padding.left + this.padding.right); //网格宽度=画布总宽度-左右间隙
        this.option.grid.height = this.height - (this.padding.top + this.padding.bottom); //网格高度=画布总宽度-上下间隙
        this.option.grid.left = -this.option.center.x + this.padding.left; //网格左侧X点
        this.option.grid.right = this.option.center.x - this.padding.right; //网格右侧X点
        this.option.grid.top = -this.option.center.y + this.padding.top; //网格顶部Y点
        this.option.grid.bottom = this.option.center.y - this.padding.bottom; //网格底部Y点

        this.option.grid.center = {
            x: (this.option.grid.width + this.padding.left - this.padding.right) / 2,
            y: (300 + this.padding.top - this.padding.bottom) / 2,
        };

        this.ctx.lineWidth = this.option.lineWidth; //线条尺寸
        this.ctx.font = this.option.fontSize + 'px 微软雅黑'; //文字
        this.ctx.fillStyle = this.option.fillStyle; //填充颜色
    }

    //绘制画布中心十字线
    private drawCenterLine() {
        this.ctx.save();
        this.ctx.translate(0, 0); //定义原始点
        this.ctx.strokeStyle = 'rgba(255,0,0,1)'; //笔触颜色
        this.ctx.beginPath(); //打开绘制路径

        this.ctx.moveTo(-this.option.center.x, 0); //X线起点
        this.ctx.lineTo(this.width, 0); //X线终点

        this.ctx.moveTo(0, -this.option.center.y); //Y线起点
        this.ctx.lineTo(0, this.height); //Y线终点

        this.ctx.stroke(); //画路径
        this.ctx.closePath(); //关闭绘制路径
        this.ctx.restore();
    }

    //绘制网格
    private drawGridLine() {
        this.ctx.save();
        this.ctx.translate(this.option.grid.left, this.option.grid.top); //定义网格原点
        this.ctx.strokeStyle = 'rgba(255,255,255,0.1)'; //笔触颜色
        let xaixsLine = (Math.abs(-180) + Math.abs(180)) / 10; //经度取值为-180~180度，每10度为1根，一共36根
        let yaixsLine = (Math.abs(-90) + Math.abs(90)) / 10; //纬度取值为-90~90度，每10度为1根，一共18根

        //以grid左上角为原点，每次递增10度画线
        for (let i = 0; i < xaixsLine + 1; i++) {
            let cellW = this.option.grid.width / xaixsLine; //每个格子宽度=网格宽度/线条数量

            this.ctx.beginPath(); //打开绘制路径
            this.ctx.moveTo(i * cellW, 0); //经线起点
            this.ctx.lineTo(i * cellW, this.option.grid.height); //经线终点

            if (i % 3 === 0) {
                //经度刻度文字
                let axisLabel = '0°';
                let deg = Math.abs(i * 10 - 180); //度数绝对值
                if (i * 10 < 180) {
                    axisLabel = deg + '°W'; //西经
                } else if (i * 10 > 180) {
                    axisLabel = deg + '°E'; //东经
                }

                let axisOffset = (axisLabel.length * this.option.fontSize) / 4; //文字居中偏移量=文字长度*字体大小/4倍 = 总文字宽度的四分之一
                this.ctx.fillText(axisLabel, i * cellW - axisOffset, this.option.grid.height + this.option.fontSize * 2); //绘制刻度,  this.option.grid.height + fontSize * 2是使文字向下偏移以免与格子重叠
                this.ctx.moveTo(i * cellW, this.option.grid.height); //经线终点
                this.ctx.arc(i * cellW, this.option.grid.height, 2, 0, 2 * Math.PI); //绘制小圆点
            }

            this.ctx.stroke(); //画路径
            this.ctx.closePath(); //关闭绘制路径
        }

        //以grid左上角为原点，每次递增10度画线
        for (let i = 0; i < yaixsLine + 1; i++) {
            let cellH = this.option.grid.height / yaixsLine; //每个格子高度=网格高度/线条数量

            this.ctx.beginPath(); //打开绘制路径
            this.ctx.moveTo(0, i * cellH); //纬线起点
            this.ctx.lineTo(this.option.grid.width, i * cellH); //纬线终点

            if (i % 3 === 0) {
                //纬度刻度文字
                let ayisLabel = '0°';
                let deg = Math.abs(i * 10 - 90); //度数绝对值
                if (i * 10 < 90) {
                    ayisLabel = deg + '°N'; //北纬
                } else if (i * 10 > 90) {
                    ayisLabel = deg + '°S'; //南纬
                }

                //纬刻度文字
                let ayisOffset = this.option.fontSize / 4;
                this.ctx.fillText(ayisLabel, -this.option.fontSize * 3, i * cellH + ayisOffset); //刻度， -this.option.fontSize * 3是使文字向左偏移以免与格子重叠
                this.ctx.moveTo(this.option.grid.width, i * cellH); //纬线终点
                this.ctx.arc(0, i * cellH, 2, 0, 2 * Math.PI); //绘制小圆点
            }

            this.ctx.stroke(); //画路径
            this.ctx.closePath(); //关闭绘制路径
        }
        this.ctx.restore();
    }

    //绘制色柱
    private drawColorColumn() {
        this.ctx.save();
        this.ctx.translate(this.option.grid.right, this.option.grid.bottom); //网格右下角
        this.ctx.fillStyle = 'gray';
        this.ctx.fillText('TECU(cm)', 10, -this.option.grid.height - this.option.fontSize * 1.2);

        let colorScale = [
            { value: 0, color: '#03a83d' },
            { value: 5, color: '#03f65a' },
            { value: 10, color: '#fae508' },
            { value: 15, color: '#f07e05' },
            { value: 20, color: '#ff1603' },
            { value: 25, color: '#ff1603' },
            { value: 30, color: '#ff0511' },
        ];

        let max = colorScale[colorScale.length - 1]['value'];
        let min = colorScale[0]['value'];

        //创建一个线性渐变
        let lg = this.ctx.createLinearGradient(0, 0, 0, -this.option.grid.height);

        let colorScaleIndex = colorScale.length - 1;

        //绘制刻度
        colorScale.forEach((item, index) => {
            this.ctx.beginPath();
            this.ctx.strokeStyle = 'gray';
            this.ctx.lineWidth = 1;
            this.ctx.moveTo(23, (-this.option.grid.height / colorScaleIndex) * index);
            this.ctx.lineTo(30, (-this.option.grid.height / colorScaleIndex) * index);

            this.ctx.fillStyle = 'gray';
            this.ctx.fillText(colorScale[index]['value'], 35, (-this.option.grid.height / colorScaleIndex) * index + this.option.fontSize / 2.5);
            this.ctx.stroke();
            this.ctx.closePath();

            lg.addColorStop((colorScale[index]['value'] - min) / (max - min), colorScale[index]['color']); //添加颜色
        });

        this.ctx.beginPath();
        this.ctx.strokeStyle = lg;
        this.ctx.lineWidth = 5;
        this.ctx.moveTo(20, 0);
        this.ctx.lineTo(20, -this.option.grid.height);
        this.ctx.stroke();
        this.ctx.closePath();
        this.ctx.restore();
    }

    //绘制点
    private drawDot() {
        let colorScale = [
            { value: 0, color: '#03a83d' },
            { value: 5, color: '#03f65a' },
            { value: 10, color: '#fae508' },
            { value: 15, color: '#f07e05' },
            { value: 20, color: '#ff1603' },
            { value: 25, color: '#ff1603' },
            { value: 30, color: '#ff0511' },
        ];

        //细分颜色值
        let colorScaleDetail: any[] = this.getColorScaleDetail(colorScale);

        //图例
        let px = 0,
            py = 0,
            x = 0,
            y = 0,
            radis = 2;

        let clickXY: any[] = [];

        let data = this.getData();
        this.ctx.save();
        this.ctx.translate((this.padding.left - this.padding.right) / 2, (this.padding.top - this.padding.bottom) / 2);

        data.forEach((item: any) => {
            x = (item.lng / 180) * (this.option.grid.width / 2);
            y = (-item.lat / 90) * (this.option.grid.height / 2);
            px = this.canvas.offsetLeft + this.option.center.x + x;
            py = this.canvas.offsetTop + this.option.center.y + y;

            clickXY.push({
                x: px,
                y: py,
                r: radis,
                d: item,
            });

            this.ctx.beginPath();
            this.ctx.fillStyle = this.getDotColor(item.value, colorScaleDetail);
            this.ctx.arc(x, y, radis, 0, 2 * Math.PI);
            this.ctx.closePath();
            this.ctx.fill();

            this.ctx.beginPath();
            this.ctx.strokeStyle = this.getDotColor(item.value, colorScaleDetail);
            this.ctx.arc(x, y, radis * 3, 0, 2 * Math.PI);
            this.ctx.stroke(); //画路径
            this.ctx.closePath();
        });
        this.ctx.restore();
    }

    //绘制背景地图
    private drawBgMap() {
        return new Promise((resolve, reject) => {
            let bgImg = require('@/assets/images/map/map-bg.png');

            const img = new Image();
            img.src = bgImg;
            img.onload = () => {
                this.ctx.drawImage(img, this.option.grid.left, this.option.grid.top, this.option.grid.width, this.option.grid.height);
                resolve(img);
            };
        });
    }

    //清除画布
    private clearCanvas() {
        this.ctx.translate(-this.option.center.x, -this.option.center.y);
        this.ctx.clearRect(0, 0, this.width, this.height);
        this.ctx.fillStyle = this.option.fillStyle; //填充颜色
        this.ctx.translate(this.option.center.x, this.option.center.y);
    }

    //细分颜色值
    private getColorScaleDetail(colorScale: any[]) {
        let max = colorScale[colorScale.length - 1]['value'];
        let min = colorScale[0]['value'];

        let colorScaleDetail: any[] = [];

        colorScale.forEach((item, index) => {
            let n = 0;
            while (n < 10 && index < colorScale.length - 1) {
                colorScaleDetail.push({
                    value: (n / 10) * (colorScale[index + 1]['value'] - colorScale[index]['value']) + colorScale[index]['value'],
                    color: this.getColorByRate(colorScale[index]['color'], colorScale[index + 1]['color'], n / 10),
                });
                n++;
            }
        });

        colorScaleDetail.push({
            value: max,
            color: colorScale[colorScale.length - 1]['color'],
        });

        return colorScaleDetail;
    }

    //获取颜色值
    private getColorByRate(startColor: any, endColor: any, rate: number) {
        let startColorRGB: string | number[] = this.color2Rgb(startColor);
        let endColorRGB: string | number[] = this.color2Rgb(endColor);
        if (Array.isArray(startColorRGB) && Array.isArray(endColorRGB)) {
            let returnColorRGB =
                'RGB(' +
                ((endColorRGB[0] - startColorRGB[0]) * rate + startColorRGB[0]).toFixed(0) +
                ',' +
                ((endColorRGB[1] - startColorRGB[1]) * rate + startColorRGB[1]).toFixed(0) +
                ',' +
                ((endColorRGB[2] - startColorRGB[2]) * rate + startColorRGB[2]).toFixed(0) +
                ')';
            return this.color2Hex(returnColorRGB);
        }

        return '#000000';
    }

    //获取点颜色
    private getDotColor(value: string, colorScaleDetail: any[]) {
        for (let j = colorScaleDetail.length - 1; j >= 0; j--) {
            if (parseFloat(value) > parseFloat(colorScaleDetail[j].value)) {
                return colorScaleDetail[j].color;
            }
        }
        return colorScaleDetail[0].color;
    }

    /*RGB颜色转换为16进制*/
    private color2Hex(rgb: string) {
        let reg = /^#([0-9a-fA-f]{3}|[0-9a-fA-f]{6})$/;
        if (/^(rgb|RGB)/.test(rgb)) {
            let aColor = rgb.replace(/(?:\(|\)|rgb|RGB)*/g, '').split(',');
            let strHex = '#';
            for (let i = 0; i < aColor.length; i++) {
                let hex = Number(aColor[i]).toString(16);
                if (hex.length < 2) {
                    hex += hex;
                }
                strHex += hex;
            }
            if (strHex.length !== 7) {
                strHex = rgb;
            }
            return strHex;
        } else if (reg.test(rgb)) {
            let aNum = rgb.replace(/#/, '').split('');
            if (aNum.length === 6) {
                return rgb;
            } else if (aNum.length === 3) {
                let numHex = '#';
                for (let i = 0; i < aNum.length; i += 1) {
                    numHex += aNum[i] + aNum[i];
                }
                return numHex;
            }
        } else {
            return rgb;
        }
    }

    /*16进制颜色转为RGB格式*/
    private color2Rgb(color: string) {
        let sColor = color.toLowerCase();
        let reg = /^#([0-9a-fA-f]{3}|[0-9a-fA-f]{6})$/;
        if (sColor && reg.test(sColor)) {
            if (sColor.length === 4) {
                let sColorNew = '#';
                for (let i = 1; i < 4; i += 1) {
                    sColorNew += sColor.slice(i, i + 1).concat(sColor.slice(i, i + 1));
                }
                sColor = sColorNew;
            }
            //处理六位的颜色值
            let sColorChange = [];
            for (let i = 1; i < 7; i += 2) {
                sColorChange.push(parseInt('0x' + sColor.slice(i, i + 2)));
            }
            return sColorChange;
        } else {
            return sColor;
        }
    }

    private getData() {
        let data: any[] = [
            { name: 'WGTN', lng: '174.811', lat: '-41.325', value: '4.84' },
            { name: 'WL01', lng: '84.912', lat: '44.418', value: '5.67' },
            { name: 'WLT1', lng: '108.507', lat: '41.568', value: '4.53' },
            { name: 'WSRT', lng: '6.605', lat: '52.916', value: '2.15' },
            { name: 'WTZR', lng: '12.879', lat: '49.146', value: '4.11' },
            { name: 'XA02', lng: '107.569', lat: '34.466', value: '4.02' },
            { name: 'XW02', lng: '104.219', lat: '26.347', value: '4.84' },
            { name: 'YAR2', lng: '115.350', lat: '-29.047', value: '4.82' },
            { name: 'YARR', lng: '115.350', lat: '-29.047', value: '2.74' },
            { name: 'ZIM2', lng: '7.465', lat: '46.878', value: '2.61' },
            { name: 'ZJ01', lng: '110.438', lat: '21.253', value: '2.35' },
            { name: 'ABMF', lng: '-61.519', lat: '16.263', value: '3.23' },
            { name: 'AIRA', lng: '130.603', lat: '31.825', value: '8.49' },
            { name: 'ALBH', lng: '-123.480', lat: '48.391', value: '5.81' },
            { name: 'ALGO', lng: '-78.063', lat: '45.957', value: '2.81' },
            { name: 'ASCG', lng: '-14.322', lat: '-7.917', value: '3.70' },
            { name: 'BAKE', lng: '-95.995', lat: '64.320', value: '7.66' },
            { name: 'BQ02', lng: '132.227', lat: '46.327', value: '3.59' },
            { name: 'BRUX', lng: '4.359', lat: '50.800', value: '3.01' },
            { name: 'CCJ2', lng: '142.199', lat: '27.068', value: '4.87' },
            { name: 'CF02', lng: '119.243', lat: '41.811', value: '2.23' },
            { name: 'CIBG', lng: '106.852', lat: '-6.491', value: '3.76' },
            { name: 'CUT0', lng: '115.898', lat: '-32.005', value: '4.40' },
            { name: 'CZ02', lng: '119.909', lat: '31.838', value: '3.17' },
            { name: 'DARW', lng: '131.137', lat: '-12.844', value: '5.15' },
            { name: 'DAV1', lng: '77.975', lat: '-68.579', value: '3.63' },
            { name: 'DH01', lng: '94.669', lat: '40.152', value: '5.44' },
            { name: 'DLF1', lng: '4.388', lat: '51.988', value: '2.90' },
            { name: 'DQ02', lng: '125.267', lat: '46.679', value: '3.98' },
            { name: 'DUND', lng: '170.602', lat: '-45.885', value: '2.54' },
            { name: 'DW01', lng: '116.977', lat: '45.509', value: '3.44' },
            { name: 'EUR2', lng: '-85.930', lat: '79.991', value: '3.83' },
            { name: 'FZ01', lng: '119.353', lat: '26.039', value: '2.77' },
            { name: 'GANP', lng: '20.324', lat: '49.036', value: '3.19' },
            { name: 'GOP6', lng: '14.786', lat: '49.915', value: '3.66' },
            { name: 'HERS', lng: '0.336', lat: '50.869', value: '1.42' },
            { name: 'HKKT', lng: '114.070', lat: '22.446', value: '1.76' },
            { name: 'HKNP', lng: '113.897', lat: '22.250', value: '2.62' },
            { name: 'HLFX', lng: '-63.603', lat: '44.685', value: '5.33' },
            { name: 'HOFN', lng: '-15.188', lat: '64.269', value: '4.57' },
            { name: 'HZ02', lng: '114.508', lat: '22.753', value: '6.14' },
            { name: 'IQAL', lng: '-68.502', lat: '63.758', value: '49.69' },
            { name: 'ISTA', lng: '29.020', lat: '41.106', value: '1.87' },
            { name: 'JOZ2', lng: '21.033', lat: '52.099', value: '2.79' },
            { name: 'KAT1', lng: '132.157', lat: '-14.376', value: '4.61' },
            { name: 'KERG', lng: '70.258', lat: '-49.353', value: '5.47' },
            { name: 'KIR8', lng: '21.061', lat: '67.880', value: '3.55' },
            { name: 'KITG', lng: '66.889', lat: '39.135', value: '4.44' },
            { name: 'KOS1', lng: '5.818', lat: '52.175', value: '2.87' },
            { name: 'KRGG', lng: '70.258', lat: '-49.353', value: '4.84' },
            { name: 'LX02', lng: '103.320', lat: '35.941', value: '3.52' },
            { name: 'MAJU', lng: '171.370', lat: '7.119', value: '13.71' },
            { name: 'MAR7', lng: '17.259', lat: '60.597', value: '3.66' },
            { name: 'MATE', lng: '16.705', lat: '40.650', value: '2.74' },
            { name: 'MAW1', lng: '62.873', lat: '-67.607', value: '2.88' },
            { name: 'MAYG', lng: '45.259', lat: '-12.782', value: '3.89' },
            { name: 'MET3', lng: '24.395', lat: '60.219', value: '5.06' },
            { name: 'METG', lng: '24.385', lat: '60.244', value: '3.05' },
            { name: 'MIZU', lng: '141.137', lat: '39.136', value: '4.85' },
            { name: 'NICO', lng: '33.397', lat: '35.142', value: '1.48' },
            { name: 'NTUS', lng: '103.683', lat: '1.346', value: '5.66' },
            { name: 'ONS1', lng: '11.925', lat: '57.397', value: '3.61' },
            { name: 'ONSA', lng: '11.926', lat: '57.397', value: '3.31' },
            { name: 'ORID', lng: '20.795', lat: '41.129', value: '2.45' },
            { name: 'PARK', lng: '148.269', lat: '-33.000', value: '5.92' },
            { name: 'PERT', lng: '115.889', lat: '-31.803', value: '5.88' },
            { name: 'PNGM', lng: '147.370', lat: '-2.043', value: '1.96' },
            { name: 'POAL', lng: '-51.111', lat: '-30.075', value: '1.24' },
            { name: 'PTGG', lng: '121.045', lat: '14.536', value: '4.86' },
            { name: 'REUN', lng: '55.573', lat: '-21.209', value: '4.78' },
            { name: 'REYK', lng: '-21.946', lat: '64.141', value: '15.09' },
            { name: 'RGDG', lng: '-67.743', lat: '-53.787', value: '3.98' },
            { name: 'RIGA', lng: '24.059', lat: '56.950', value: '3.02' },
            { name: 'SASK', lng: '-106.391', lat: '52.198', value: '3.67' },
            { name: 'SAVO', lng: '-38.423', lat: '-12.940', value: '4.67' },
            { name: 'SCH2', lng: '-66.824', lat: '54.834', value: '3.36' },
            { name: 'SCTB', lng: '166.763', lat: '-77.851', value: '3.38' },
            { name: 'SEYG', lng: '55.532', lat: '-4.679', value: '5.85' },
            { name: 'SGPO', lng: '-97.477', lat: '36.605', value: '2.60' },
            { name: 'SIN1', lng: '103.682', lat: '1.343', value: '6.14' },
            { name: 'SOD3', lng: '26.390', lat: '67.423', value: '4.12' },
            { name: 'SPT0', lng: '12.892', lat: '57.717', value: '2.84' },
            { name: 'STJO', lng: '-52.669', lat: '47.597', value: '2.45' },
            { name: 'STK2', lng: '141.849', lat: '43.530', value: '5.54' },
            { name: 'SZ04', lng: '113.915', lat: '22.522', value: '2.61' },
            { name: 'THTG', lng: '-149.600', lat: '-17.578', value: '3.53' },
            { name: 'TJ01', lng: '117.724', lat: '38.981', value: '2.25' },
            { name: 'TLSE', lng: '1.481', lat: '43.562', value: '2.61' },
            { name: 'TOW2', lng: '147.060', lat: '-19.270', value: '3.80' },
            { name: 'UFPR', lng: '-49.222', lat: '-25.449', value: '1.82' },
            { name: 'URUM', lng: '87.603', lat: '43.809', value: '49.22' },
            { name: 'UTQI', lng: '-156.609', lat: '71.325', value: '7.80' },
            { name: 'VALD', lng: '-77.556', lat: '48.098', value: '6.07' },
            { name: 'VIS0', lng: '18.368', lat: '57.656', value: '3.73' },
        ];

        let randomData = data.map((item: any) => {
            item.value = parseFloat(item.value) + (Math.random() - 0.5) * 10;
            return item;
        });

        return randomData;
    }
}
</script>

<style lang="scss" scoped>
.canvas {
    background-image: url('~@/assets/images/map/map-bg.png');
    background-repeat: no-repeat;
}
</style>
