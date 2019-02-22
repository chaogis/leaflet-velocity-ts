# leaflet-velocity-ts [![NPM version][npm-image]][npm-url]

This is a typescript updated version of [leaflet-velocity](https://github.com/danwild/leaflet-velocity)

### Compared to the other version :
* This version is compatible with the latest version of leaflet.
* This version adds a better particles management when zooming and moving the map which gives better performance on mobile devices.
* This version does not need jQuery in order to work.
* This version does not need leaflet-velocity.css to be included.

## Example use:
```javascript
var velocityLayer = L.velocityLayer({

	displayValues: true,
	displayOptions: {
		velocityType: 'Global Wind',
		position: 'bottomleft',//REQUIRED !
		emptyString: 'No velocity data',//REQUIRED !
		angleConvention: 'bearingCW',//REQUIRED !
		displayPosition: 'bottomleft',
		displayEmptyString: 'No velocity data',
		speedUnit: 'm/s'
	},
	data: data,            // see demo/*.json, or wind-js-server for example data service

	// OPTIONAL
	/*minVelocity: 0,      // used to align color scale
	maxVelocity: 10,       // used to align color scale*/
	velocityScale: 0.005,  // modifier for particle animations, arbitrarily defaults to 0.005
	colorScale: []         // define your own array of hex/rgb colors
});

velocityLayer.addTo(mymap);
```
## License
MIT License (MIT)


[npm-image]: https://badge.fury.io/js/leaflet-velocity-ts.svg
[npm-url]: https://www.npmjs.com/package/leaflet-velocity-ts

## 针对不同坐标系的使用
参考 https://github.com/danwild/leaflet-velocity/issues/15
	具体内容如下：
	Hi @danwild! Thanks for your answer. I discovered modifying "lat" variable where "invert(x, y, windy)" is defined to, it do the trick:
	_lat = rad2deg(windy.north) + y / windy.height * rad2deg(mapLatDelta);_
	with _var mapLatDelta = windy.south - windy.north;_

	You can visualize the results fine until zoom 6 with Web Mercator projection. At zoom 5 image canvas just don't adjust to map. However, with EPSG4326 looks good at all zoom levels and this is what I want, to get native projection from satellite and another meteorology products.

	And I was close to forget it, variable "H" should be modified too:

	before // _var H = Math.pow(10, -5.2);_
	After // _var H = 5;_

	I attach a zip file with your main demo but modified to be able to use in your computer. If you have any quastion, please don't hesitate to ask me. Thanks for your awesome plugin.

## 针对真气网风场数据的解码
v = e.getGzipData = function (t, e, n, i)
        {
            var r = l.
        default.windhost;
            void 0 === t && (t = (0, d.getSmpFormatNowDate)());
            var o = "aqi" === e || "pm2_5" === e || "co" === e ? 0:
            -8,
                s = (0, d.getFormatDateHourAdd)(t, o, "yyyyMMddhh") + "00",
                u = s.substr(0, 8),
                c = "0.25";
            if (n < 5 && "aqi" !== e)
                {
                    for (var h = s.substr(8, 4), f = ["0000", "0300", "0600", "0900", "1200", "1500", "1800", "2100"], p = 0; p < f.length; p++)
                    {
                        if (h >= f[p] && h < f[p + 1])
                        {
                            h = f[p];
                            break
                        }
                        if (p === f.length - 1)
                        {
                            h = f[p];
                            break
                        }
                    }
                    s = u + h,
                    c = "1.0"
                }
            r += "gzip/" + u + "/" + c + "/" + s + "-" + e + "-surface-level-gfs-" + c + ".json",
            fetch(r, {
                    headers: {
                        Accept: "text/plain",
                        "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8"
                    },
                    method: "GET"
                }).then(function (t)
                {
                    return t.text()
                }).then(function (t)
                {
                    var e = a.
                default.inflate(t, {
                        to: "string"
                    }),
                        n = JSON.parse(e);
                    i(n)
                }).
            catch (function (t)
                {
                    console.log("其他异常:", t)
                })
        },
	
# 使用pako进行解码，参考 https://github.com/nodeca/pako
# 对真气网数据的加载使用，展示上需要修改leaflet-velocity的源码，如下：
var o = this.createBuilder(t),
                            a = o.header,
                            s = a.lo1, //西经
                            l = a.la1 > a.la2 ? a.la1 : a.la2, //北纬，实际取la2
                            u = a.dx, //网格宽度
                            c = a.dy, //网格高度
                            d = a.nx, //宽度
                            h = a.ny, //高度
                            f = new Date(a.refTime);
                        f.setHours(f.getHours() + a.forecastTime);
                        var p = [], //对应grid
                            m = 0, //对应p
                            g = Math.floor(d * u) >= 360; //对应isContinuous
                        if (a.la1 > a.la2) for (var v = 0; v < h; v++)
                            {
                                for (var y = [], _ = 0; _ < d; _++, m++) y[_] = o.data(m);
                                g && y.push(y[0]),
                                p[v] = y
                            }
                        else for (var b = h - 1; b >= 0; b--)
                            {
                                for (var x = [], w = 0; w < d; w++, m++) x[w] = o.data(m);
                                g && x.push(x[0]),
                                p[b] = x
                            }
                        t = null,
                        e(
                            {
                                date: f,
                                interpolate: r_
                            })
# 需要对la1 la2的大小进行判断，并据此对外层循环进行处理
