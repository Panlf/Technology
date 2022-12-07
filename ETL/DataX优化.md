# DataX使用优化
## 关键参数
- job.setting.speed.channel channel并发数
- job.setting.speed.record 全局配置channel的record限速
- job.setting.speed.byte 全局配置channel的byte限速
- core.transport.channel.speed.record 单个channel的record限速
- core.transport.channel.speed.byte 单个channel的byte限速

## 提升每个channel的速度
在DataX内部对每个Channel会有严格的速度控制，分两种，一种是控制每秒同步的记录数，另一种是每秒同步的字节数，默认的速度限制是1MB/s，可以根据具体硬件情况设置这个byte速度或者record速度，一般设置byte速度，比如：我们可以把单个Channel的速度上限配置为5MB

## 提升DataX Job内Channel并发数
并发数 = taskGroup的数量 * 每个TaskGroup 并发执行的Task数（默认为5）

提升job内Channel并发有三种配置方式

### 配置全局Byte限速以及单Channel Byte限速

Channel个数 = 全局Byte限速 + 单Channel Byte限速

```
{
	"core":{
		"transport":{
			"channel":{
				"speed":{
					"byte":1048576
				}
			}
		}
	},
	"job":{
		"setting":{
			"speed":{
				"byte":5242880
			}
		}
	}
	
}
```

### 配置全局Record限速以及单Channel Record限速
Channel个数 = 全局Record限速 / 单Channel Record限速

```
{
	"core":{
		"transport":{
			"channel":{
				"speed":{
					"record":100
				}
			}
		}
	},
	"job":{
		"setting":{
			"speed":{
				"record":500
			}
		}
	}
}
```

### 直接配置Channel个数
```
{
	"job":{
		"setting":{
			"speed":{
				"channel":5
			}
		}
	}
}
```


## 提高JVM堆内存
当提升DataX Job内Channel并发数时，内存的占用会显著增加，因为DataX作为数据交换通道，在内存中会缓存较多的数据。例如Channel中会有一个Buffer，作为临时数据交换的缓冲区，而在部分Reader和Writer的中，也会存在一些Buffer，为了防止OOM等错误，调大JVM堆内存。

建议将内存设置为4G或者8G，这个也可以根据实际情况来调整。

调整JVM参数
```
python datax/bin/datax.py --jvm="-Xms8G -Xmx8G" xxx.json
```