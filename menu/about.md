---
layout: page
---

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>
<script src='https://d3js.org/d3.v5.min.js'></script>

<div id='blank' style='height:10px;'></div>

<div id='container'>
<svg style='width:347px;height:350px;letter-spacing:10px'>
	<g></g>
</svg>
</div>

<script>
var svg = d3.select('#container').select('svg');

var points = [
	[116, 209],
	[118, 180],
	[122, 156],
	[127, 147],
	[130, 144],
	[135, 141],
	[139, 152],
	[150, 149],
	[162, 151],
	[166, 151],
	[180, 149],
	[185, 150],
	[181, 144],
	[165, 142],
	[157, 140],
	[150, 144],
	[151, 132],
	[153, 130],
	[160, 129],
	[174, 129],
	[180, 128],
	[186, 127],
	[199, 126],
	[215, 137],
	[231, 142],
	[239, 145],
	[232, 138],
	[236, 135],
	[244, 139],
	[247, 136],
	[251, 138],
	[244, 141],
	[239, 145],
	[252, 150],
	[265, 151],
	[265, 150],
	[272, 156],
	[275, 156],
	[279, 158],
	[282, 155],
	[291, 160],
	[299, 160],
	[310, 180],
	[311, 190],
	[311, 211],
	[307, 234],
	[306, 235],
	[305, 256],
	[304, 264],
	[300, 276],
	[298, 279],
	[281, 299],
	[258, 314],
	[247, 325],
	[248, 326],
	[268, 330],
	[278, 328],
	[282, 332],
	[273, 350],
	[275, 347],
	[283, 336],
	[290, 310],
	[294, 297],
	[304, 282],
	[311, 249],
	[313, 236],
	[321, 235],
	[324, 229],
	[327, 221],
	[339, 189],
	[340, 182],
	[340, 165],
	[343, 152],
	[344, 137],
	[347, 116],
	[344, 102],
	[336, 77],
	[310, 40],
	[290, 23],
	[264, 11],
	[246, 4],
	[229, 3],
	[208, 0],
	[198, 0],
	[190, 6],
	[182, 5],
	[167, 8],
	[159, 12],
	[148, 18],
	[133, 30],
	[120, 42],
	[109, 52],
	[106, 60],
	[106, 65],
	[90, 97],
	[89, 103],
	[88, 111],
	[90, 129],
	[92, 149],
	[92, 155],
	[95, 161],
	[97, 166],
	[104, 177],
	[106, 180],
	[110, 193],
	[115, 206],
	[116, 209],

	[-1, -1],

	[138, 135],
	[142, 138],
	[141, 143],
	[138, 135],
	
	[-1, -1],

	[328, 209],
	[333, 191],
	
	[-1, -1],

	[319, 219],
	[325, 220],
	[323, 226],
	[319, 219],
	
	[-1, -1],

	[172, 172],
	[180, 174],
	[185, 178],
	[189, 185],
	[179, 190],
	[181, 187],
	[178, 183],
	[170, 184],
	[167, 181],
	[164, 175],
	[159, 177],
	[160, 181],
	[155, 182],
	[147, 180],
	[149, 178],
	[172, 172],
	
	[-1, -1],

	[181, 176],
	[184, 180],
	[178, 183],
	[181, 176],
	
	[-1, -1],

	[249, 174],
	[255, 172],
	[274, 174],
	[284, 176],
	[288, 180],
	[286, 184],
	[278, 188],
	[273, 184],
	[265, 185],
	[254, 184],
	[249, 177],
	[246, 180],
	[247, 184],
	[243, 186],
	[242, 184],
	[249, 174],
	
	[-1, -1],

	[266, 175],
	[265, 182],
	[269, 177],
	[266, 175],

	[-1, -1],

	[200, 236],
	[205, 239],
	[197, 241],
	[200, 236],
	
	[-1, -1],

	[219, 243],
	[228, 235],
	[233, 236],
	[236, 238],
	[243, 234],
	[230, 240],
	[226, 243],
	[219, 243],
	
	[-1, -1],

	[182, 267],
	[192, 266],
	[210, 267],
	[223, 266],
	[232, 265],
	[239, 266],
	[252, 265],
	[255, 268],
	[248, 274],
	[227, 283],
	[234, 276],
	[227, 272],
	[217, 272],
	[187, 271],
	[177, 270],
	[182, 267],
];

var scale = 1.0;

var len = points.length;

var minx = points[0][0];
var miny = points[0][1];
var maxx = points[0][0];
var maxy = points[0][1];

for (var i = 0; i < len; i++) {
	if (minx > points[i][0] && points[i][0] >= 0) minx = points[i][0];
	if (maxx < points[i][0]) maxx = points[i][0];
	if (miny > points[i][1] && points[i][1] >= 0) miny = points[i][1];
	if (maxy < points[i][1]) maxy = points[i][1];
}

console.log('min', [minx, miny]);
console.log('max', [maxx, maxy]);

var i = 0;	
move();

function move() {
	if (i >= len-1) {
		console.log('end of drawing');
		return;
	}

	if (points[i][0] < 0 || points[(i+1)%len][0] < 0) {
		i++;
		move();
		return;
	}
	
	var shift_x = -minx;
	var shift_y = -miny;
	var cur_point_x = points[i][0] + shift_x;
	var cur_point_y = points[i][1] + shift_y;
	var next_point_x = points[(i+1)%len][0] + shift_x;
	var next_point_y = points[(i+1)%len][1] + shift_y;
		
	var line = svg.append('line')
		.attr('class', function(d) {
			return 'solid';
		})
		.attr('stroke', 'black')
		.attr('x1', cur_point_x * scale)
		.attr('y1', cur_point_y * scale)
		.attr('x2', cur_point_x * scale)
		.attr('y2', cur_point_y * scale);

	
	line.attr('x1', cur_point_x * scale)
		.attr('y1', cur_point_y * scale)
		.transition()
		.duration(10)
		.attr('x2', next_point_x * scale)
		.attr('y2', next_point_y * scale)
		.transition()
		.duration(10)
		.on('start', move);

	i++;
}

</script>

### 호 종 현
Jonghyun Ho

Software Engineer

Interested in Graphics, Robot, Cloud Computing, Data Analysis, AI, Software Architecture, Quant, ...

