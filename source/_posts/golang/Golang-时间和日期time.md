---
title: Golang-时间和日期time
date: 2018-11-15 10:33:06
tags: [Golang]
categories: [Golang]
toc: true
cover: '/images/categories/golang.jpeg'
---

time包提供了时间的显示和测量用的函数。日历的计算采用的是公历。

# type **ParseError**
```go
`ParseError` 描述解析时间字符串时出现的错误。
type ParseError struct {
    Layout     string
    Value      string
    LayoutElem string
    ValueElem  string
    Message    string
}
```

## func **(\*ParseError) Error**
`Error` 返回 `ParseError` 的字符串表示。

# type **Weekday**
`Weekday` 代表一周的某一天。
```go
type Weekday int

const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)
```

## func **(Weekday) String**
`String` 返回该日（周几）的英文名（"Sunday"、"Monday"，……）
```go
func (d Weekday) String() string
```
示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	fmt.Println(time.Weekday(2).String())
}
```
输出：

    Tuesday

# type **Month**
`Month` 代表一年的某个月。
```go
type Month int

const (
    January Month = 1 + iota
    February
    March
    April
    May
    June
    July
    August
    September
    October
    November
    December
)
```
示例：
```go
_, month, day := time.Now().Date()
if month == time.November && day == 10 {
    fmt.Println("Happy Go day!")
}
```

## func **(Month) String**
```go
func (m Month) String() string
```
`String` 返回月份的英文名（"January"，"February"，……）

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	fmt.Println(time.Month(10).String())
}
```
输出：

    October

# type Location
```go
type Location struct {
    name string
	zone []zone
	tx   []zoneTrans
	cacheStart int64
	cacheEnd   int64
	cacheZone  *zone
}
```
`Location` 代表一个（关联到某个时间点的）地点，以及该地点所在的时区。
```go
var Local *Location = &localLoc
```
`Local` 代表系统本地，对应本地时区。
```go
var UTC *Location = &utcLoc
```
`UTC` 代表通用协调时间，对应零时区。

## func **LoadLocation**
```go
func LoadLocation(name string) (*Location, error)
```
`LoadLocation` 返回使用给定的名字创建的 `Location`。

如果 `name` 是""或"UTC"，返回 `UTC`；如果 `name` 是"Local"，返回 `Local`；否则 `name` 应该是IANA时区数据库里有记录的地点名（该数据库记录了地点和对应的时区），如"America/New_York"。

`LoadLocation` 函数需要的时区数据库可能不是所有系统都提供，特别是非Unix系统。此时 `LoadLocation` 会查找环境变量 `ZONEINFO` 指定目录或解压该变量指定的zip文件（如果有该环境变量）；然后查找Unix系统的惯例时区数据安装位置，最后查找`$GOROOT/lib/time/zoneinfo.zip`。

## func **FixedZone**
```go
func FixedZone(name string, offset int) *Location
```
`FixedZone` 使用给定的地点名 `name` 和时间偏移量 `offset`（单位秒）创建并返回一个`Location`


## func **(\*Location) String**
```go
func (l *Location) String() string
```
`String` 返回对时区信息的描述，返回值绑定为 `LoadLocation` 或 `FixedZone` 函数创建 `l` 时的 `name` 参数。

## 示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	location, err := time.LoadLocation("Local")
	fmt.Println(location.String(), err)

	fixed := time.FixedZone("ZH", 3600 * 8)
	fmt.Println(fixed.String())
}
```
输出：

    Local <nil>
    ZH

# type Time
```go
type Time struct {
    wall uint64
    ext  int64
	loc *Location
}
```
`Time` 代表一个纳秒精度的时间点。

程序中应使用 `Time` 类型值来保存和传递时间，而不能用指针。就是说，表示时间的变量和字段，应为 `time.Time` 类型，而不是 `*time.Time.`类型。一个Time类型值可以被多个 go 程同时使用。时间点可以使用Before、After和Equal方法进行比较。`Sub` 方法让两个时间点相减，生成一个 `Duration` 类型值（代表时间段）。`Add` 方法给一个时间点加上一个时间段，生成一个新的 `Time` 类型时间点。

`Time` 零值代表时间点January 1, year 1, 00:00:00.000000000 UTC。因为本时间点一般不会出现在使用中，`IsZero` 方法提供了检验时间是否显式初始化的一个简单途径。

每一个时间都具有一个地点信息（及对应地点的时区信息），当计算时间的表示格式时，如 `Format`、`Hour` 和 `Year` 等方法，都会考虑该信息。`Local`、`UTC` 和 `In` 方法返回一个指定时区（但指向同一时间点）的 `Time`。修改地点/时区信息只是会改变其表示；不会修改被表示的时间点，因此也不会影响其计算。

## func **Date**
```go
func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time
```
`Date` 返回一个时区为loc、当地时间为：

    year-month-day hour:min:sec + nsec nanoseconds

的时间点。

month、day、hour、min、sec和nsec的值可能会超出它们的正常范围，在转换前函数会自动将之规范化。如October 32被修正为November 1。

夏时制的时区切换会跳过或重复时间。如，在美国，March 13, 2011 2:15am从来不会出现，而November 6, 2011 1:15am 会出现两次。此时，时区的选择和时间是没有良好定义的。Date会返回在时区切换的两个时区其中一个时区正确的时间，但本函数不会保证在哪一个时区正确。

如果loc为nil会panic。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	t := time.Date(2018, time.November, 15, 15, 0, 0, 0, time.Local)
	fmt.Printf("Go launched at %s\n", t.Local())
}
```
输出：

    Go launched at 2018-11-15 15:00:00 +0800 CST

## func **Now**
```go
func Now() Time
```
`Now` 返回当前本地时间。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    fmt.Println(time.Now())
}
```
输出：

    2018-11-15 15:11:48.29489808 +0800 CST m=+0.000180466

## func **Parse**
```go
func Parse(layout, value string) (Time, error)
```
`Parse` 解析一个格式化的时间字符串并返回它代表的时间。layout定义了参考时间：

    Mon Jan 2 15:04:05 -0700 MST 2006

在输入格式下的字符串表示，作为输入的格式的示例。同样的格式规则会被用于输入字符串。

预定义的ANSIC、UnixDate、RFC3339和其他版式描述了参考时间的标准或便捷表示。要获得更多参考时间的定义和格式，参见本包的ANSIC和其他版式常量。

`value` 中漏掉的元素会被视为0；如果不能是0，会被视为1。因此，解析"3:04pm"会返回对应时间点：Jan 1, year 0, 15:04:00 UTC的Time（注意因为year为0，该时间在Time零值之前）。年份必须在0000..9999范围内。周几会被检查其语法，但是会被忽略。

如果缺少表示时区的信息，`Parse` 会将时区设置为 `UTC`。

当解析具有时区偏移量的时间字符串时，如果该时区偏移量和本地时区相同，`Parse`会在返回值中将 `Location` 设置为本地和本地时区。否则，它会将 `Location` 设置为一个虚构的具有该时区偏移量的值。

当解析具有时区缩写的时间字符串时，如果该时区缩写具有已定义的时间偏移量，会使用该偏移量。如果时区缩写是"UTC"，会将该时间视为 `UTC` 时间，不考虑 `Location`。如果时区缩写是未知的，`Parse` 会将 `Location` 设置为一个虚构的地点为时区缩写，时间偏移量为0的值。这种做法是为了让一个时间可以在同一版式下不丢失信息的被解析和重新格式化；但字符串表示和具体表示的时间点会因为实际时区偏移量而不同。为了避免这些问题，请使用数字表示的时区偏移量，或者使用 `ParseInLocation` 函数。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	// longForm 通过示例展示了引用时间如何在期望的布局中表示。
	const longForm = "Jan 2, 2006 at 3:04pm (MST)"
	t, _ := time.Parse(longForm, "Feb 3, 2013 at 7:54pm (PST)")
    fmt.Println(t)
    
	// 短格式是另一种将参考时间表示在所需布局中的方式;它没有时区。
    // 注意:没有显式区域，返回UTC时间。
	const shortForm = "2006-Jan-02"
	t, _ = time.Parse(shortForm, "2013-Feb-03")
	fmt.Println(t)
}
```
输出：

    2013-02-03 19:54:00 +0000 PST
    2013-02-03 00:00:00 +0000 UTC

## func **ParseInLocation**
```go
func ParseInLocation(layout, value string, loc *Location) (Time, error)
```
`ParseInLocation` 类似 `Parse` 但有两个重要的不同之处。第一，当缺少时区信息时，`Parse` 将时间解释为 `UTC` 时间，而 `ParseInLocation` 将返回值的 `Location` 设置为 `loc` ；第二，当时间字符串提供了时区偏移量信息时，`Parse` 会尝试去匹配本地时区，而 `ParseInLocation` 会去匹配 `loc`。
示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	loc, _ := time.LoadLocation("Europe/Berlin")
	const longForm = "Jan 2, 2006 at 3:04pm (MST)"
	t, _ := time.ParseInLocation(longForm, "Jul 9, 2012 at 5:02am (CEST)", loc)
	fmt.Println(t)
	// 注意:没有显式区域，返回给定位置的时间。
	const shortForm = "2006-Jan-02"
	t, _ = time.ParseInLocation(shortForm, "2012-Jul-09", loc)
	fmt.Println(t)
}
```
输出：

    2012-07-09 05:02:00 +0200 CEST
    2012-07-09 00:00:00 +0200 CEST

## func **Unix**
```go
func Unix(sec int64, nsec int64) Time
```
`Unix` 创建一个本地时间，对应 `sec` 和 `nsec` 表示的Unix时间（从January 1, 1970 UTC至该时间的 *秒数* 和 *纳秒数*）。

nsec的值在[0, 999999999]范围外是合法的。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

	fmt.Println(now)
	fmt.Println(now.Unix())
}
```
输出：

    2018-11-15 15:43:14.481928482 +0800 CST m=+0.000165517
    1542267794

## func **(Time) Zone**
```go
func (t Time) Zone() (name string, offset int)
```
`Zone` 计算 `t` 所在的时区，返回该时区的规范名（如"CET"）和该时区相对于 `UTC` 的时间偏移量（单位秒）。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Zone())
}
```
输出：

    2018-11-15 15:45:55.254153367 +0800 CST m=+0.000163920
    CST 28800

## func **(Time) IsZero**
```go
func (t Time) IsZero() bool
```
`IsZero` 报告 `t` 是否代表 `Time` 零值的时间点，January 1, year 1, 00:00:00 UTC。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.IsZero())
}
```
输出：

    2018-11-15 15:55:24.159958577 +0800 CST m=+0.000204054
    false

## func **(Time) Local**
```go
func (t Time) Local() Time
```
`Local` 返回采用本地和本地时区，但指向同一时间点的 `Time`。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

	fmt.Println(now)
	fmt.Println(now.Local())
}
```
输出：

    2018-11-15 15:57:15.790800933 +0800 CST m=+0.000254212
    2018-11-15 15:57:15.790928159 +0800 CST

## func **Location**
```go
func (t Time) Location() *Location
```
`Location` 返回 `t` 的地点和时区信息。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Location())
}
```
输出：

    2018-11-15 15:45:55.254153367 +0800 CST m=+0.000163920
    Local

## func **(Time) UTC**
```go
func (t Time) UTC() Time
```
`UTC` 返回采用UTC和零时区，但指向同一时间点的Time。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.UTC())
}
```
输出：

    2018-11-15 16:08:18.626081249 +0800 CST m=+0.000231182
    2018-11-15 08:08:18.626081249 +0000 UTC

## unc **(Time) In**
```go
func (t Time) In(loc *Location) Time
```
`In` 返回采用 `loc` 指定的地点和时区，但指向同一时间点的 `Time`。如果 `loc` 为 `nil` 会 `panic`。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.In(now.Location()))
}
```
输出：

    2018-11-15 16:14:25.710321361 +0800 CST m=+0.000169674
    2018-11-15 16:14:25.710321361 +0800 CST

## func **(Time) Unix**
```go
func (t Time) Unix() int64
```
`Unix` 将 `t` 表示为Unix时间，即从时间点January 1, 1970 UTC到时间点t所经过的时间（单位秒）。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Unix())
}
```
输出：

    2018-11-15 16:28:08.284601879 +0800 CST m=+0.000149523
    1542270488

## func **(Time) UnixNano**
```go
func (t Time) UnixNano() int64
```
`UnixNano` 将 `t` 表示为Unix时间，即从时间点January 1, 1970 UTC到时间点 `t` 所经过的时间（单位 *纳秒*）。如果纳秒为单位的 `unix` 时间超出了 `int64` 能表示的范围，结果是未定义的。注意这就意味着 `Time` 零值调用 `UnixNano` 方法的话，结果是未定义的。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.UnixNano())
}
```
输出：

    2018-11-15 16:47:12.069698257 +0800 CST m=+0.000168169
    1542271632069698257

## func **(Time) Equal**
```go
func (t Time) Equal(u Time) bool
```
判断两个时间是否相同，会考虑时区的影响，因此不同时区标准的时间也可以正确比较。本方法和用t==u不同，这种方法还会比较地点和时区信息。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Equal(time.Now()))
}
```
输出：

    2018-11-15 16:50:00.212637423 +0800 CST m=+0.000195819
    false

## func **(Time) Before**
```go
func (t Time) Before(u Time) bool
```
如果 `t` 代表的时间点在 `u` 之前，返回真；否则返回假。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Before(time.Now()))
}
```
输出：

    2018-11-16 13:33:34.139887944 +0800 CST m=+0.000173372
    true

## func **(Time) After**
```go
func (t Time) After(u Time) bool
```
如果 `t` 代表的时间点在 `u` 之后，返回真；否则返回假。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.After(time.Now()))
}
```
输出：

    2018-11-16 13:34:33.919810923 +0800 CST m=+0.000241874
    false

## func **(Time) Date**
```go
func (t Time) Date() (year int, month Month, day int)
```
返回时间点 `t` 对应的年、月、日。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Date())
}
```
输出：

    2018-11-16 13:37:03.060141712 +0800 CST m=+0.002025614
    2018 November 16

## func **(Time) Clock**
```go
func (t Time) Clock() (hour, min, sec int)
```
返回 `t` 对应的那一天的时、分、秒。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Date())
	fmt.Println(now.Clock())
}
```
输出：

    2018-11-16 13:39:08.904053286 +0800 CST m=+0.000157283
    2018 November 16
    13 39 8

## func **(Time) Year**
```go
func (t Time) Year() int
```
返回时间点 `t` 对应的年份。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Year())
}
```
输出：

    2018-11-16 13:39:08.904053286 +0800 CST m=+0.000157283
    2018


## func **(Time) Month**
```go
func (t Time) Month() Month
```
返回时间点 `t` 对应那一年的第几月。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Month())
}
```
输出：

2018-11-16 13:43:04.629329432 +0800 CST m=+0.000132658
November

## func **(Time) ISOWeek**
```go
func (t Time) ISOWeek() (year, week int)
```
返回时间点 `t` 对应的 `ISO 9601` 标准下的年份和星期编号。星期编号范围[1,53]，1月1号到1月3号可能属于上一年的最后一周，12月29号到12月31号可能属于下一年的第一周。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.ISOWeek())
}
```
输出：

    2018-11-16 13:45:21.793130986 +0800 CST m=+0.000151798
    2018 46     // 2018年第46个星期

## func **(Time) YearDay**
```go
func (t Time) YearDay() int
```
返回时间点 `t` 对应的那一年的*第几天*，平年的返回值范围[1,365]，闰年[1,366]。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.YearDay())
}
```
输出：

    2018-11-16 13:49:50.793130986 +0800 CST m=+0.000151798
    320

## func **(Time) Day**
```go
func (t Time) Day() int
```
返回时间点 `t` 对应那一月的第几日。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Day())
}
```
输出：

    2018-11-16 13:50:11.793130986 +0800 CST m=+0.000151798
    16

## func **(Time) Weekday**
```go
func (t Time) Weekday() Weekday
```
返回时间点 `t` 对应的那一周的周几。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Weekday())
}
```
输出：

    2018-11-16 13:51:23.793130986 +0800 CST m=+0.000151798
    Friday

## func **(Time) Hour**
```go
func (t Time) Hour() int
```
返回 `t` 对应的那一天的第几小时，范围[0, 23]。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Hour())
}
```
输出：

    2018-11-16 13:52:00.793130986 +0800 CST m=+0.000151798
    13

## func **(Time) Minute**
```go
func (t Time) Minute() int
```
返回 `t` 对应的那一小时的第几分种，范围[0, 59]。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Minute())
}
```
输出：

    2018-11-16 13:53:25.793130986 +0800 CST m=+0.000151798
    53

## func **(Time) Second**
```go
func (t Time) Second() int
```
返回 `t` 对应的那一分钟的第几秒，范围[0, 59]。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Second())
}
```
输出：

    2018-11-16 13:54:50.793130986 +0800 CST m=+0.000151798
    50

## func **(Time) Nanosecond**
```go
func (t Time) Nanosecond() int
```
返回 `t` 对应的那一秒内的*纳秒偏移量*，范围[0, 999999999]。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Nanosecond())
}
```
输出：

    2018-11-16 13:57:35.502361392 +0800 CST m=+0.001316926
    502361392

## func **(Time) Add**
```go
func (t Time) Add(d Duration) Time
```
`Add` 返回时间点 `t+d`。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.Add(1e9))   // 纳秒
}
```
输出：

    2018-11-16 14:03:22.029219211 +0800 CST m=+0.000157850
    2018-11-16 14:03:23.029219211 +0800 CST m=+1.000157850

## func **(Time) AddDate**
```go
func (t Time) AddDate(years int, months int, days int) Time
```
`AddDate` 返回增加了给出的年份、月份和天数的时间点 `Time`。例如，时间点January 1, 2011调用 `AddDate(-1, 2, 3)` 会返回March 4, 2010。

`AddDate` 会将结果规范化，类似 `Date` 函数的做法。因此，举个例子，给时间点October 31添加一个月，会生成时间点December 1。（从时间点November 31规范化而来）

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.AddDate(1, 1, 1))
}
```
输出：

    2018-11-16 14:07:25.009463355 +0800 CST m=+0.000165232
    2019-12-17 14:07:25.009463355 +0800 CST

## func **(Time) Sub**
```go
func (t Time) Sub(u Time) Duration
```
返回一个时间段 `t-u`。如果结果超出了 `Duration` 可以表示的最大值/最小值，将返回最大值/最小值。要获取时间点 `t-d`（d为Duration），可以使用t.Add(-d)。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

	fmt.Println(now)
	fmt.Println(time.Now().Sub(now))
}
```
输出：

    2018-11-16 14:22:55.865289 +0800 CST m=+0.000171211
    65.876µs    // 微秒

## func **(Time) Round**
```go
func (t Time) Round(d Duration) Time
```
返回距离 `t` 最近的时间点，该时间点应该满足从 `Time` 零值到该时间点的时间段能整除 `d`；如果有两个满足要求的时间点，距离 `t` 相同，会向上舍入；如果 `d <= 0`，会返回 `t` 的拷贝。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	t := time.Date(0, 0, 0, 12, 15, 30, 918273645, time.UTC)
	round := []time.Duration{
		time.Nanosecond,
		time.Microsecond,
		time.Millisecond,
		time.Second,
		2 * time.Second,
		time.Minute,
		10 * time.Minute,
		time.Hour,
	}
	for _, d := range round {
		fmt.Printf("t.Round(%6s) = %s\n", d, t.Round(d).Format("15:04:05.999999999"))
	}
}
```
输出：

    t.Round(   1ns) = 12:15:30.918273645
    t.Round(   1µs) = 12:15:30.918274
    t.Round(   1ms) = 12:15:30.918
    t.Round(    1s) = 12:15:31
    t.Round(    2s) = 12:15:30
    t.Round(  1m0s) = 12:16:00
    t.Round( 10m0s) = 12:20:00
    t.Round(1h0m0s) = 12:00:00

## func **(Time) Truncate**
```go
func (t Time) Truncate(d Duration) Time
```
类似 `Round`，但是返回的是最接近但早于 `t` 的时间点；如果 `d <= 0`，会返回t的拷贝。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
   t, _ := time.Parse("2006 Jan 02 15:04:05", "2012 Dec 07 12:15:30.918273645")
    trunc := []time.Duration{
        time.Nanosecond,
        time.Microsecond,
        time.Millisecond,
        time.Second,
        2 * time.Second,
        time.Minute,
        10 * time.Minute,
        time.Hour,
    }
    for _, d := range trunc {
        fmt.Printf("t.Truncate(%6s) = %s\n", d, t.Truncate(d).Format("15:04:05.999999999"))
    }
}
```
输出：

    t.Truncate(   1ns) = 12:15:30.918273645
    t.Truncate(   1us) = 12:15:30.918273
    t.Truncate(   1ms) = 12:15:30.918
    t.Truncate(    1s) = 12:15:30
    t.Truncate(    2s) = 12:15:30
    t.Truncate(  1m0s) = 12:15:00
    t.Truncate( 10m0s) = 12:10:00
    t.Truncate(1h0m0s) = 12:00:00

## func **(Time) Format**
```go
func (t Time) Format(layout string) string
```
`Format` 根据 `layout` 指定的格式返回 `t` 代表的时间点的格式化文本表示。`layout` 定义了参考时间：

    Mon Jan 2 15:04:05 -0700 MST 2006

格式化后的字符串表示，它作为期望输出的例子。同样的格式规则会被用于格式化时间。

预定义的ANSIC、UnixDate、RFC3339和其他版式描述了参考时间的标准或便捷表示。要获得更多参考时间的定义和格式，参见本包的ANSIC和其他版式常量。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    // layout shows by example how the reference time should be represented.
	const layout = "Jan 2, 2006 at 3:04pm (MST)"
	t := time.Date(2009, time.November, 10, 15, 0, 0, 0, time.Local)
	fmt.Println(t.Format(layout))
    fmt.Println(t.UTC().Format(layout))
    
    fmt.Println(time.Now().Format("2006-01-02 15:04:05")) // 2006-01-02 15:04:05
}
```
输出：

    Nov 10, 2009 at 3:00pm (CST)
    Nov 10, 2009 at 7:00am (UTC)
    2018-11-16 14:40:17

## func **(Time) String**
```go
func (t Time) String() string
```
`String` 返回采用如下格式字符串的格式化时间。

    "2006-01-02 15:04:05.999999999 -0700 MST"

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
    now := time.Now()

    fmt.Println(now)
	fmt.Println(now.String())
}
```
输出：

    2018-11-16 14:50:11.257346871 +0800 CST m=+0.000177186
    2018-11-16 14:50:11.257346871 +0800 CST m=+0.000247204

## func **(Time) GobEncode**
```go
func (t Time) GobEncode() ([]byte, error)
```
`GobEncode` 实现了 `gob.GobEncoder` 接口。

## func **(Time) GobDecode**
```go
func (t *Time) GobDecode(data []byte) error
```
`GobDecode` 实现了 `gob.GobDecoder` 接口。

## func **(Time) MarshalBinary**
```go
func (t Time) MarshalBinary() ([]byte, error)
```
`MarshalBinary` 实现了 `encoding.BinaryMarshaler` 接口。

## func **(Time) UnmarshalBinary**
```go
func (t *Time) UnmarshalBinary(data []byte) error
```
`UnmarshalBinary` 实现了 `encoding.BinaryUnmarshaler` 接口。

## func **(Time) MarshalJSON**
```go
func (t Time) MarshalJSON() ([]byte, error)
```
`MarshalJSON` 实现了 `json.Marshaler` 接口。返回值是用双引号括起来的采用RFC 3339格式进行格式化的时间表示，如果需要会提供小于秒的精度。

## func **(Time) UnmarshalJSON**
```go
func (t *Time) UnmarshalJSON(data []byte) (err error)
```
`UnmarshalJSON` 实现了 `json.Unmarshaler` 接口。时间被期望是双引号括起来的RFC 3339格式。

## func **(Time) MarshalText**
```go
func (t Time) MarshalText() ([]byte, error)
```
`MarshalText` 实现了 `encoding.TextMarshaler` 接口。返回值是采用RFC 3339格式进行格式化的时间表示，如果需要会提供小于秒的精度。

## func **(Time) UnmarshalText**
```go
func (t *Time) UnmarshalText(data []byte) (err error)
```
`UnmarshalText` 实现了 `encoding.TextUnmarshaler` 接口。时间被期望采用RFC 3339格式。

# type **Duration**
```go
type Duration int64
```
`Duration` 类型代表两个时间点之间经过的时间，以纳秒为单位。可表示的最长时间段大约290年。
```go
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```
常用的时间段。没有定义一天或超过一天的单元，以避免夏时制的时区切换的混乱。

要将 `Duration` 类型值表示为某时间单元的个数，用除法：
```go
second := time.Second
fmt.Print(int64(second/time.Millisecond)) // prints 1000
```
要将整数个某时间单元表示为 `Duration` 类型值，用乘法：
```go
seconds := 10
fmt.Print(time.Duration(seconds)*time.Second) // prints 10s
```
示例：
```go
t0 := time.Now()
expensiveCall()
t1 := time.Now()
fmt.Printf("The call took %v to run.\n", t1.Sub(t0))
```

## func **ParseDuration**
```go
func ParseDuration(s string) (Duration, error)
```
`ParseDuration` 解析一个时间段字符串。一个时间段字符串是一个序列，每个片段包含可选的正负号、十进制数、可选的小数部分和单位后缀，如"300ms"、"-1.5h"、"2h45m"。合法的单位有"ns"、"us" /"µs"、"ms"、"s"、"m"、"h"。

## func **Since**
```go
func Since(t Time) Duration
```
`Since` 返回从 `t` 到现在经过的时间，等价于time.Now().Sub(t)。

## func **(Duration) Hours**
```go
func (d Duration) Hours() float64
```
`Hours` 将时间段表示为 `float64` 类型的小时数。

## func **(Duration) Minutes**
```go
func (d Duration) Minutes() float64
```
`Minutes` 将时间段表示为 `float64` 类型的分钟数。

## func **(Duration) Seconds**
```go
func (d Duration) Seconds() float64
```
`Seconds` 将时间段表示为 `float64` 类型的秒数。

## func **(Duration) Nanoseconds**
```go
func (d Duration) Nanoseconds() int64
```
`Nanoseconds` 将时间段表示为 `int64` 类型的纳秒数，等价于int64(d)。

## func **(Duration) String**
```go
func (d Duration) String() string
```
返回时间段采用"72h3m0.5s"格式的字符串表示。最前面可以有符号，数字+单位为一个单元，开始部分的0值单元会被省略；如果时间段<1s，会使用"ms"、"us"、"ns"来保证第一个单元的数字不是0；如果时间段为0，会返回"0"。


# type **Timer**
```go
type Timer struct {
    C <-chan Time
    // 内含隐藏或非导出字段
}
```
`Timer` 类型代表单次时间事件。当 `Timer` 到期时，当时的时间会被发送给 `C`，除非 `Timer` 是被 `AfterFunc` 函数创建的。

## func **NewTimer**
```go
func NewTimer(d Duration) *Timer
```
`NewTimer` 创建一个 `Timer`，它会在最少过去时间段 `d` 后到期，向其自身的 `C`字段发送当时的时间。

## func **AfterFunc**
```go
func AfterFunc(d Duration, f func()) *Timer
```
`AfterFunc` 另起一个 `go` 进程等待时间段 `d` 过去，然后调用 `f`。它返回一个 `Timer`，可以通过调用其 `Stop` 方法来取消等待和对f的调用。


## func **(\*Timer) Reset**
```go
func (t *Timer) Reset(d Duration) bool
```
`Reset` 使 `t` 重新开始计时，（本方法返回后再）等待时间段d过去后到期。如果调用时 `t` 还在等待中会返回真；如果 `t` 已经到期或者被停止了会返回假。

## func **(\*Timer) Stop**
```go
func (t *Timer) Stop() bool
```
`Stop` 停止 `Timer` 的执行。如果停止了 `t` 会返回真；如果 `t` 已经被停止或者过期了会返回假。`Stop` 不会关闭通道 `t.C`，以避免从该通道的读取不正确的成功。


# type **Ticker**
```go
type Ticker struct {
    C <-chan Time // 周期性传递时间信息的通道
    // 内含隐藏或非导出字段
}
```
`Ticker` 保管一个通道，并每隔一段时间向其传递"tick"。

## func **NewTicker**
```go
func NewTicker(d Duration) *Ticker
```
`NewTicker` 返回一个新的 `Ticker`，该 `Ticker` 包含一个通道字段，并会每隔时间段 `d` 就向该通道发送当时的时间。它会调整时间间隔或者丢弃 `tick` 信息以适应反应慢的接收者。如果d<=0会panic。关闭该 `Ticker` 可以释放相关资源。

## func **(\*Ticker) Stop**
```go
func (t *Ticker) Stop()
```
`Stop` 关闭一个 `Ticker`。在关闭后，将不会发送更多的tick信息。`Stop` 不会关闭通道 `t.C`，以避免从该通道的读取不正确的成功。

## func **Sleep**
```go
func Sleep(d Duration)
```
`Sleep` 阻塞当前 `go` 进程至少 `d` 代表的时间段。d<=0时，`Sleep` 会立刻返回。

示例：
```go
package main

import (
	"fmt"
	"time"
)

func main()  {
	//t0 := time.Now()

	t1 := time.Now()
	fmt.Println(t1)

	time.Sleep(5 * 1000 * time.Millisecond)

	fmt.Println(time.Now())
}
```
输出：

    2018-11-16 15:48:29.089446905 +0800 CST m=+0.000154708
    2018-11-16 15:48:34.089593643 +0800 CST m=+5.000301421  // 两次输出间隔 5 秒

## func **After**
```go
func After(d Duration) <-chan Time
```
`After` 会在另一线程经过时间段 `d` 后向返回值发送当时的时间。等价于NewTimer(d).C。

示例：
```go
select {
case m := <-c:
    handle(m)
case <-time.After(5 * time.Minute):
    fmt.Println("timed out")
}
```

## func **Tick**
```go
func Tick(d Duration) <-chan Time
```
`Tick` 是 `NewTicker` 的封装，只提供对 `Ticker` 的通道的访问。如果不需要关闭 `Ticker`，本函数就很方便。

示例：
```go
c := time.Tick(1 * time.Minute)
for now := range c {
    fmt.Printf("%v %s\n", now, statusUpdate())
}
```
