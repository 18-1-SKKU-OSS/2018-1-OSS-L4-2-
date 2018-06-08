Z-Wave 예제
==============

아래는 많은 일반적인 명령어들과 분석이 된 이벤트들의 예를 보여주는 Device Handler 코드 샘플입니다. 

GitHub인 `여기 <https://github.com/SmartThingsCommunity/Code/blob/master/device-types/z-wave-example.groovy>`__ 에서 이 예제를 볼 수 있습니다. 

.. code-block:: groovy

	metadata {
		definition (name: "Z-Wave Device Reference", author: "SmartThings") {
			capability "Actuator"
			capability "Switch"
			capability "Polling"
			capability "Refresh"
			capability "Temperature Measurement"
			capability "Sensor"
			capability "Battery"
		}

		simulator {
			// 이것들은 당신의 device handler로 이벤트 메세지를 보내는걸 테스트 하기 위해 
			// IDE simulator에 나타납니다
			status "basic report on": 
			       zwave.basicV1.basicReport(value:0xFF).incomingMessage()
			status "basic report off": 
			        zwave.basicV1.basicReport(value:0).incomingMessage()
			status "dimmer switch on at 70%": 
			       zwave.switchMultilevelV1.switchMultilevelReport(value:70).incomingMessage()
			status "basic set on":
				   zwave.basicV1.basicSet(value:0xFF).incomingMessage()
			status "temperature report 70°F":
					zwave.sensorMultilevelV2.sensorMultilevelReport(scaledSensorValue: 70.0, precision: 1, sensorType: 1, scale: 1).incomingMessage()
			status "low battery alert": 
			       zwave.batteryV1.batteryReport(batteryLevel:0xFF).incomingMessage()
			status "multichannel sensor": 
			       zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1).encapsulate(zwave.sensorBinaryV1.sensorBinaryReport(sensorValue:0)).incomingMessage()

			// 켜기 실행            
			reply "2001FF,delay 5000,2002": "command: 2503, payload: FF"

			// 끄기 실행
			reply "200100,delay 5000,2002": "command: 2503, payload: 00"
		}

		tiles {
			standardTile("switch", "device.switch", width: 2, height: 2,
			            canChangeIcon: true) {
				state "on", label: '${name}', action: "switch.off", 
				      icon: "st.unknown.zwave.device", backgroundColor: "#79b821"
				state "off", label: '${name}', action: "switch.on", 
				      icon: "st.unknown.zwave.device", backgroundColor: "#ffffff"
			}
			standardTile("refresh", "command.refresh", inactiveLabel: false, 
			             decoration: "flat") {
				state "default", label:'', action:"refresh.refresh", 
				      icon:"st.secondary.refresh"
			}
			
			valueTile("battery", "device.battery", inactiveLabel: false, 
			          decoration: "flat") {
				state "battery", label:'${currentValue}% battery', unit:""
			}

			valueTile("temperature", "device.temperature") {
				state("temperature", label:'${currentValue}°',
					backgroundColors:[
						[value: 31, color: "#153591"],
						[value: 44, color: "#1e9cbb"],
						[value: 59, color: "#90d2a7"],
						[value: 74, color: "#44b621"],
						[value: 84, color: "#f1d801"],
						[value: 95, color: "#d04e00"],
						[value: 96, color: "#bc2323"]
					]
				)
			}

			main (["switch", "temperature"])
			details (["switch", "temperature", "refresh", "battery"])
		}
	}

	def parse(String description) {
		def result = null
		def cmd = zwave.parse(description, [0x60: 3])
		if (cmd) {
			result = zwaveEvent(cmd)
			log.debug "Parsed ${cmd} to ${result.inspect()}"
		} else {
			log.debug "Non-parsed event: ${description}"
		}
		result
	}

	def zwaveEvent(physicalgraph.zwave.commands.basicv1.BasicReport cmd)
	{
		def result = []
		result << createEvent(name:"switch", value: cmd.value ? "on" : "off")

		// 다중 레벨 스위치를 위해, cmd.value는 1부터 99까지의 값을 나타낼 수 있습니다 
		// 조광 레벨
		result << createEvent(name:"level", value: cmd.value, unit:"%", 
		            descriptionText:"${device.displayName} dimmed ${cmd.value==255 ? 100 : cmd.value}%")

		result
	}

	def zwaveEvent(physicalgraph.zwave.commands.switchbinaryv1.SwitchBinaryReport cmd) {
		createEvent(name:"switch", value: cmd.value ? "on" : "off")
	}
	
	def zwaveEvent(physicalgraph.zwave.commands.switchmultilevelv3.SwitchMultilevelReport cmd) {
		def result = []
		result << createEvent(name:"switch", value: cmd.value ? "on" : "off")
		result << createEvent(name:"level", value: cmd.value, unit:"%", 
		             descriptionText:"${device.displayName} dimmed ${cmd.value==255 ? 100 : cmd.value}%")
		result
	}

	def zwaveEvent(physicalgraph.zwave.commands.meterv1.MeterReport cmd) {
		def result
		if (cmd.scale == 0) {
			result = createEvent(name: "energy", value: cmd.scaledMeterValue, 
			                     unit: "kWh")
		} else if (cmd.scale == 1) {
			result = createEvent(name: "energy", value: cmd.scaledMeterValue, 
			                     unit: "kVAh")
		} else {
			result = createEvent(name: "power", 
			                     value: Math.round(cmd.scaledMeterValue), unit: "W")
		}

		result
	}

	def zwaveEvent(physicalgraph.zwave.commands.meterv3.MeterReport cmd) {
		def map = null
		if (cmd.meterType == 1) {
			if (cmd.scale == 0) {
				map = [name: "energy", value: cmd.scaledMeterValue, 
				       unit: "kWh"]
			} else if (cmd.scale == 1) {
				map = [name: "energy", value: cmd.scaledMeterValue, 
				       unit: "kVAh"]
			} else if (cmd.scale == 2) {
				map = [name: "power", value: cmd.scaledMeterValue, unit: "W"]
			} else {
				map = [name: "electric", value: cmd.scaledMeterValue]
				map.unit = ["pulses", "V", "A", "R/Z", ""][cmd.scale - 3]
			}
		} else if (cmd.meterType == 2) {
			map = [name: "gas", value: cmd.scaledMeterValue]
			map.unit =	["m^3", "ft^3", "", "pulses", ""][cmd.scale]
		} else if (cmd.meterType == 3) {
			map = [name: "water", value: cmd.scaledMeterValue]
			map.unit = ["m^3", "ft^3", "gal"][cmd.scale]
		}
		if (map) {
			if (cmd.previousMeterValue && cmd.previousMeterValue != cmd.meterValue) {
				map.descriptionText = "${device.displayName} ${map.name} is ${map.value} ${map.unit}, previous: ${cmd.scaledPreviousMeterValue}"
			}
			createEvent(map)
		} else {
			null
		}
	}

	def zwaveEvent(physicalgraph.zwave.commands.sensorbinaryv2.SensorBinaryReport cmd) {
		def result
		switch (cmd.sensorType) {
			case 2:
				result = createEvent(name:"smoke", 
					value: cmd.sensorValue ? "detected" : "closed")
				break
			case 3:
				result = createEvent(name:"carbonMonoxide", 
					value: cmd.sensorValue ? "detected" : "clear")
				break
			case 4:
				result = createEvent(name:"carbonDioxide", 
					value: cmd.sensorValue ? "detected" : "clear")
				break
			case 5:
				result = createEvent(name:"temperature", 
					value: cmd.sensorValue ? "overheated" : "normal")
				break
			case 6:
				result = createEvent(name:"water", 
					value: cmd.sensorValue ? "wet" : "dry")
				break
			case 7:
				result = createEvent(name:"temperature", 
					value: cmd.sensorValue ? "freezing" : "normal")
				break
			case 8:
				result = createEvent(name:"tamper", 
					value: cmd.sensorValue ? "detected" : "okay")
				break
			case 9:
				result = createEvent(name:"aux", 
					value: cmd.sensorValue ? "active" : "inactive")
				break
			case 0x0A:
				result = createEvent(name:"contact", 
					value: cmd.sensorValue ? "open" : "closed")
				break
			case 0x0B:
				result = createEvent(name:"tilt", value: cmd.sensorValue ? "detected" : "okay")
				break
			case 0x0C:
				result = createEvent(name:"motion", 
					value: cmd.sensorValue ? "active" : "inactive")
				break
			case 0x0D:
				result = createEvent(name:"glassBreak", 
					value: cmd.sensorValue ? "detected" : "okay")
				break
			default:
				result = createEvent(name:"sensor", 
					value: cmd.sensorValue ? "active" : "inactive")
				break
		}
		result
	}

	def zwaveEvent(physicalgraph.zwave.commands.sensorbinaryv1.SensorBinaryReport cmd)
	{
		// SensorBinary의 version 1은 센서 타입이 없습니다
		createEvent(name:"sensor", value: cmd.sensorValue ? "active" : "inactive")
	}

	def zwaveEvent(physicalgraph.zwave.commands.sensormultilevelv5.SensorMultilevelReport cmd)
	{
		def map = [ displayed: true, value: cmd.scaledSensorValue.toString() ]
		switch (cmd.sensorType) {
			case 1:
				map.name = "temperature"
				map.unit = cmd.scale == 1 ? "F" : "C"
				break;
			case 2:
				map.name = "value"
				map.unit = cmd.scale == 1 ? "%" : ""
				break;
			case 3:
				map.name = "illuminance"
				map.value = cmd.scaledSensorValue.toInteger().toString()
				map.unit = "lux"
				break;
			case 4:
				map.name = "power"
				map.unit = cmd.scale == 1 ? "Btu/h" : "W"
				break;
			case 5:
				map.name = "humidity"
				map.value = cmd.scaledSensorValue.toInteger().toString()
				map.unit = cmd.scale == 0 ? "%" : ""
				break;
			case 6:
				map.name = "velocity"
				map.unit = cmd.scale == 1 ? "mph" : "m/s"
				break;
			case 8:
			case 9:
				map.name = "pressure"
				map.unit = cmd.scale == 1 ? "inHg" : "kPa"
				break;
			case 0xE:
				map.name = "weight"
				map.unit = cmd.scale == 1 ? "lbs" : "kg"
				break;
			case 0xF:
				map.name = "voltage"
				map.unit = cmd.scale == 1 ? "mV" : "V"
				break;
			case 0x10:
				map.name = "current"
				map.unit = cmd.scale == 1 ? "mA" : "A"
				break;
			case 0x12:
				map.name = "air flow"
				map.unit = cmd.scale == 1 ? "cfm" : "m^3/h"
				break;
			case 0x1E:
				map.name = "loudness"
				map.unit = cmd.scale == 1 ? "dBA" : "dB"
				break;
		}
		createEvent(map)
	}

	// 많은 센서가 BasicSet명령을 관련 장치에 전송합니다 
	// 이를 통해 스위치 유형의 장치와 연결할 수 있고,
	// 센서가 트리거 되면 직접 켜거나 끌 수 있습니다.
	def zwaveEvent(physicalgraph.zwave.commands.basicv1.BasicSet cmd)
	{
		createEvent(name:"sensor", value: cmd.value ? "active" : "inactive")
	}

	def zwaveEvent(physicalgraph.zwave.commands.batteryv1.BatteryReport cmd) {
		def map = [ name: "battery", unit: "%" ]
		if (cmd.batteryLevel == 0xFF) {	 // Special value for low battery alert
			map.value = 1
			map.descriptionText = "${device.displayName} has a low battery"
			map.isStateChange = true
		} else {
			map.value = cmd.batteryLevel
		}
		// 마지막 배터리 업데이트 시간을 저장하여 절전 모드가 해제될 때마다 묻지 않도록 합니다.
		state.lastbatt = new Date().time
		createEvent(map)
	}

	// 배터리 구동 장치는 주기적으로 작동하거나 체크인 하도록 구성할 수 있습니다.
	// 그들은 이 명령을 보내고, 다시 명령들을 받을 수 있을 만큼 충분히 깨어 있습니다. 
	// 혹은, 그들에게 더 이상 수신할 명령이 없다고 가르쳐줄 WakeUpNoMoreInformation명령을 받아
	// 더 이상 듣지 않을 수 있도록 할 수 있습니다.
	def zwaveEvent(physicalgraph.zwave.commands.wakeupv2.WakeUpNotification cmd)
	{
		def result = [createEvent(descriptionText: "${device.displayName} woke up", isStateChange: false)]

		// BatterReport가 한동안 없는 경우에만 배터리를 요청하세요.
		if (!state.lastbatt || (new Date().time) - state.lastbatt > 24*60*60*1000) {
			result << response(zwave.batteryV1.batteryGet())
			result << response("delay 1200")  // leave time for device to respond to batteryGet
		}
		result << response(zwave.wakeUpV1.wakeUpNoMoreInformation())
		result
	}

	def zwaveEvent(physicalgraph.zwave.commands.associationv2.AssociationReport cmd) {
		def result = []
		if (cmd.nodeId.any { it == zwaveHubNodeId }) {
			result << createEvent(descriptionText: "$device.displayName is associated in group ${cmd.groupingIdentifier}")
		} else if (cmd.groupingIdentifier == 1) {
			// 그룹 1애 적절하게 속해있지 않기 때문에, 관계를 세팅해 주세요. 
			result << createEvent(descriptionText: "Associating $device.displayName in group ${cmd.groupingIdentifier}")
			result << response(zwave.associationV1.associationSet(groupingIdentifier:cmd.groupingIdentifier, nodeId:zwaveHubNodeId))
		}
		result
	}

	// Security명령 클래스를 지원하는 장치는암호화된 형식의 메세지를 보낼 수 있습니다.
	// 그들은 SecurityMessageEncapsulation 명령으로 감싸 보내지며, 반드시 암호화를 풀어야 합니다.
	def zwaveEvent(physicalgraph.zwave.commands.securityv1.SecurityMessageEncapsulation cmd) {
		def encapsulatedCommand = cmd.encapsulatedCommand([0x98: 1, 0x20: 1]) 

		// 여기서는 zwave.parse에서 했던 것처럼, 명령 클래스의 버젼을 지정할 수 있습니다. 
		if (encapsulatedCommand) {
			return zwaveEvent(encapsulatedCommand)
		}
	}

	// MultiChanneldEncap및 MultySnstanceCmdEncap는 
	// 메시지가 구별할 수 없는 여러 하위 메뉴 또는"엔드 포인트"중 
	// 하나에서 온 것임을 나타낼 수 있는 방법입니다.
	def zwaveEvent(physicalgraph.zwave.commands.multichannelv3.MultiChannelCmdEncap cmd) {
		def encapsulatedCommand = cmd.encapsulatedCommand([0x30: 1, 0x31: 1]) 

		// 여기서는 zwave.parse에서 했던 것처럼, 명령 클래스의 버젼을 지정할 수 있습니다. 
		log.debug ("Command from endpoint ${cmd.sourceEndPoint}: ${encapsulatedCommand}")
		
		if (encapsulatedCommand) {
			return zwaveEvent(encapsulatedCommand)
		}
	}

	def zwaveEvent(physicalgraph.zwave.commands.multichannelv3.MultiInstanceCmdEncap cmd) {
		def encapsulatedCommand = cmd.encapsulatedCommand([0x30: 1, 0x31: 1]) 

		// 여기서는 zwave.parse에서 했던 것처럼, 명령 클래스의 버젼을 지정할 수 있습니다. 
		log.debug ("Command from instance ${cmd.instance}: ${encapsulatedCommand}")
		
		if (encapsulatedCommand) {
			return zwaveEvent(encapsulatedCommand)
		}
	}

	def zwaveEvent(physicalgraph.zwave.Command cmd) {
		createEvent(descriptionText: "${device.displayName}: ${cmd}")
	}

	def on() {
		delayBetween([
			zwave.basicV1.basicSet(value: 0xFF).format(),
			zwave.basicV1.basicGet().format()
		], 5000)  //점진적으로 변화하는 디머의 경우 5초 지연이 발생하지 않고 바로 전환할 수 있습니다.
	}

	def off() {
		delayBetween([
			zwave.basicV1.basicSet(value: 0x00).format(),
			zwave.basicV1.basicGet().format()
		], 5000)  // 점진적으로 변화하는 디머의 경우 5초 지연이 발생하지 않고 바로 전환할 수 있습니다.
	}

	def refresh() {
		// Get 명령어의 몇가지 예시들 
		delayBetween([
			zwave.switchBinaryV1.switchBinaryGet().format(),
			zwave.switchMultilevelV1.switchMultilevelGet().format(),
			zwave.meterV2.meterGet(scale: 0).format(),	// get kWh
			zwave.meterV2.meterGet(scale: 2).format(),	// get Watts
			zwave.sensorMultilevelV1.sensorMultilevelGet().format(),
			zwave.sensorMultilevelV5.sensorMultilevelGet(sensorType:1, scale:1).format(),  // get temp in Fahrenheit
			zwave.batteryV1.batteryGet().format(),
			zwave.basicV1.basicGet().format(),
		], 1200)
	}

	// 장치 유형에 폴링 기능을 추가하면 이 명령이 약 5분마다 호출되어 장치 상태를 확인합니다.

	def poll() {
		zwave.basicV1.basicGet().format()
	}

	//장치 유형에 구성 기능을 추가하면 장치가 가입한 직후에 이 명령을 호출하여 장치별 구성 명령을 설정합니다.

	def configure() {
		delayBetween([
			//configurationSet.size는 1,2또는 4이며 일반적으로 구성에서 장치가 사용하는 크기와 일치해야 합니다.
			zwave.configurationV1.configurationSet(parameterNumber:1, size:2, scaledConfigurationValue:100).format(),
			
			// zwaveHubNodeId 변수는 장치의 연결에 허브를 추가하는 데 사용할 수 있습니다.
			zwave.associationV1.associationSet(groupingIdentifier:2, nodeId:zwaveHubNodeId).format(),
			
			// 배터리 구동 센서가 4시간마다 WakeUpNotifications 를 허브로 전송하는지 확인하십시오.
			zwave.wakeUpV1.wakeUpIntervalSet(seconds:4 * 3600, nodeid:zwaveHubNodeId).format(),
		])
	} 
