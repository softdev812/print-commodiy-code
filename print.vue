<template>
	<div id="printCanvas2">
		<GridLayout :layout="printData"
				:style="{'margin-top': -marginH + 'px'}"
				:is-draggable="false"
				:is-resizable="false"
				:autoSize="true"
				:row-height=cellH
				:col-num="4"
				:margin="[marginW, marginH]"
				:use-css-transforms="true">
			<GridItem v-for="item in printData"
					style="text-align: center; border: solid 1px gray; color: black;"
					:static="item.static"
					:key="item.traceCode"
					:x="item.x"
					:y="item.y"
					:w="1"
					:h="1"
					:i="item.traceCode">
						<!-- <div style="margin-top: 15px; font-weight: bold;" :style="{'font-size': titleSize + 'px'}">IceWing</div> -->
						<vue-qr class="vue-qr" :text="item.traceCode" :size="qrH" :style="{'margin-top': marginTop + 'px'}"></vue-qr>
						<div :style="{'font-size': nameSize + 'px'}">{{item.name}}</div>
						<div style="letter-spacing:-2px;" :style="{'font-size': codeSize + 'px'}">{{item.asnReference1}}</div>
				</GridItem>
		</GridLayout>
	</div>
</template>

<script>
  import VueQr from 'vue-qr'
	import VueGridLayout from 'vue-grid-layout'
	import Print from 'avue-plugin-print'

	const PAPER_WIDTH = 1000;
	const PAPER_HIGHT = 293.6;
	const PAPER_INDENT = 0;

	const BASE_SCREEN_WIDTH_SIZE = 1920;
	const BASE_MARGIN_TOP = 60;
	const BASE_FONT_SIZE_TITLE = 36;
	const BASE_FONT_SIZE_NAME = 30;
	const BASE_FONT_SIZE_CODE = 28;

	export default {
    components: {
      GridLayout: VueGridLayout.GridLayout,
      GridItem: VueGridLayout.GridItem,
			VueQr: VueQr
    },
    data() {
			return {
				windowWidth: document.documentElement.clientWidth - 40,  // adjust screen height
				cellH: 0,
				marginW: 0,
				marginH:0,
				marginTop: 0,
				qrH: 300,
				titleSize: BASE_FONT_SIZE_TITLE,
				nameSize: BASE_FONT_SIZE_NAME,
				codeSize: BASE_FONT_SIZE_CODE,
				printData: []
			}
		},
		watch: {
      windowWidth (val) {
        let that = this;
        console.log("实时屏幕宽度：",val, that.windowHeight );
      }
		},
		mounted() {
			var that = this;
			window.onresize = () => {
				return (() => {
					window.fullWidth = document.documentElement.clientWidth;
					that.windowWidth = window.fullWidth;
					this.cellH = this.windowWidth * PAPER_HIGHT / PAPER_WIDTH;
					// this.marginW = this.windowWidth * PAPER_INDENT / PAPER_WIDTH;
					this.marginH = this.windowWidth * PAPER_INDENT / PAPER_WIDTH;
				})()
			};
		},
		created() {
			this.cellH = this.windowWidth * PAPER_HIGHT / PAPER_WIDTH;
			this.qrH = this.cellH * 0.64;
			// this.marginW = this.windowWidth * PAPER_INDENT / PAPER_WIDTH;
			this.marginH = this.windowWidth * PAPER_INDENT / PAPER_WIDTH;
			this.marginTop = BASE_MARGIN_TOP * this.windowWidth / BASE_SCREEN_WIDTH_SIZE;
			this.titleSize = BASE_FONT_SIZE_TITLE * this.windowWidth / BASE_SCREEN_WIDTH_SIZE;
			this.nameSize = BASE_FONT_SIZE_NAME * this.windowWidth / BASE_SCREEN_WIDTH_SIZE;
			this.codeSize = BASE_FONT_SIZE_CODE * this.windowWidth / BASE_SCREEN_WIDTH_SIZE;
			console.log("size：", this.windowWidth, this.cellH);
		},
		methods: {
			init (data) {
				this.printData = data
				// Open the print window
				const w = window.open()

				var that = this
				setTimeout(function() {
					that.printContent(w)
				},2000);
			},
			printContent(w) {				
				// Get HTML to print from element
        const prtHtml = document.getElementById('printCanvas2').innerHTML;

        // Get all stylesheets HTML
        let stylesHtml = '';
        for (const node of [...document.querySelectorAll('link[rel="stylesheet"], style')]) {
          stylesHtml += node.outerHTML;
        }

        w.document.write(`<!DOCTYPE html>
          <html>
            <head>
              ${stylesHtml}
            </head>
            <body>
              ${prtHtml}
            </body>
          </html>`);				

				// w.document.body.innerHTML = prtHtml;

        w.addEventListener('beforeprint', (e)=> {
          console.log('beforePrint');
        });

        w.addEventListener('afterprint', (e)=> {
					console.log('afterPrint');
					this.$emit('callbackComplete')
        });

        w.document.close();
        w.focus();
        w.print();

        w.close();
			}
		}
	}
</script>

<style>
#printCanvas2 {
  visibility: hidden;
}
.grid-item {
  text-align: center;
  border: solid 1px gray;
	color: black;
  /* width: 240px;
  height: 240px; */
  /* padding: 5px; */
}
.grid-item .title {
	margin-top: 10px;
  font-weight: bold;
}
.grid-item .detail{
  font-size: 25px;
  letter-spacing:-4px
}

@media print
{
	#printCanvas2 {
		visibility: visible;
	}
	.grid-item {
		text-align: center;
		color: black;
		/* border: none; */
		/* width: 240px;
		height: 240px; */
		/* padding: 5px; */
	}
}
</style>