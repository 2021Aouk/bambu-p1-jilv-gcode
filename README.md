# bambu-p1-jilv-gcode
拓竹p1急律gcode，不磨嘴、无清理线、精简步骤、快速开打、0维护
;原理是打完再清理喷嘴，节省掉下次开打清理的时间，所以首次运行前擦干净喷嘴！！
;首次擦不干净喷嘴为正常现象，试个两三次
;pei纹理板记得补偿zoffset（拓竹切片没有，用orca）
;开始和结束g代码都在txt里，分别复制进切片
;如果你是从别人手上拿到本文件，强烈建议看一遍b站视频：BV1ddS2YREu6
;交流群：522185942，欢迎反馈问题、合作开发~
;开发者注意！注释过多会使g代码不执行
;开始g代码
 
M220 S100 ;速度100%
M221 S100 ;流量100%

M140 S[bed_temperature_initial_layer_single] ;设置热床温度，不等待
M104 S{nozzle_temperature_initial_layer[initial_extruder]+10} ; 喷嘴比设定高10度，调平
M17 X1.2 Y1.2 Z0.75 ;将电机电流复位为默认值
M17 Z0.4 ; 启用电机(降低Z马达电流)

M1002 set_gcode_claim_speed_level : 5
M221 X0 Y0 Z0 ;关闭软端限位，防止潜在的逻辑问题
G29.1 Z{+0.0} ; 首先清除z-trim值
M204 S10000 ; init ACC set to 10m/s^2




;g28xy后再g28z非要在右手近门处挪一点点，G4 P1000 ;等待1秒也没有用，g91也没用，M290 X256 Y0 Z2.6666666 ;设个零点也不行，电机电流M17 X1.2 Y1.2 Z0.4也不行，应该是拓竹的safez，如果使用g91记得把这一段算上！！ ;将m290设在g28前会导致g28也参考m290，后续所有任务傻b竹子死活归零无力，而我找不到竹子的失能g代码，只能重启，浪费我一晚上，cao
;将m290设在g28后也会导致后续所有任务傻b竹子死活归零无力

G28 X Y 
G1 X250 Y0 F30000

G28 Z P0 T300; home z with low precision,permit 300 temperature
M104 S{nozzle_temperature_initial_layer[initial_extruder]+0} ; z归零后立马回温
G1 X246 Y0 Z-0.25
G91
;因为没上料所以此处挤出没有用，如果打废或者手动进料会挤出，记得清理
G1 X10 Y0 Z0 E5 F18000
G1 X-5 Y0 Z5 E6 
G1 X-5 Y0 Z-5 E6
G1 X10 Y0 Z0 E6
G1 Z5
G90



M17 X1.2 Y1.2 Z0.75 ;将电机电流复位为默认值
;M17 x1.2 y1.2 z0.75；将电机电流复位为默认值，设高会不会更强
M975 S1 ; 打开振动补偿

;G1 X80 Y245 F30000 ;移动到马桶前面
;G1 Y266 F3000 ;慢慢移到马桶里，y267就可能撞丢步，虽然再往里废料更容易进去
;注释掉以上2个代码，拓竹ams代码里会自己回坑的

;以下三个g代码停顿了好一会儿，可优化吗
M1002 gcode_claim_action : 2
M106 P1 S0 ;关风扇
G29.2 S0 ;关闭ABL（自动调平）

M620 M ;AMS,好像只要用ams就会自己回厕所往前再往后
M620 S[initial_extruder]A   ; 如果AMS存在，则切换材料
    M109 S[nozzle_temperature_initial_layer] ;热端升至打印温度
   T[initial_extruder] ；注释掉会不挤出,调试用
    M400
M621 S[initial_extruder]AM621 S [initial_extruder]
;设置在最初温度下启用ams？
M620.1 E F{filament_max_volumetric_speed[initial_extruder]/2.4053*60} T{nozzle_temperature_range_high[initial_extruder]}
;进料后，挤出设置为最大体积速度/耗材截面积*60，在最高的 设置的喷嘴温度下

M412 S1 ; 打开断料检检测===
G1 X60 Y245 F25000  ;冗余代码：注释掉同时关闭ams会导致喷嘴在上文（这里是归零）坐标挤出
G1 X84 Y266 F25000  ;冗余代码：防止ams不执行，挤出烧塑料
M104 S[nozzle_temperature_initial_layer] ;冗余代码：热端回至打印温度，准备冲刷
G92 E0
G1 E5 Z0.2 F3000；换料不干净别增加这个，
;貌似e大了不执行
G92 E0
G1 E190 F1000 ；换料不干净增加这个E
；换料不干净增加这个EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE
；换料不干净增加这个EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE
；换料不干净增加这个E,pla顶petg：200没问题，170，180底面扣三丝，petg顶pla175，185扯三丝
G92 E0
G1 X145 F25000 ;丢步可能是y太靠后
G1 E0.01 F3600;挤出一点防止开打缺料 

;G1 X145 F25000   ； g1 x145 f25000
;没擦干净带出来是挤出太少重力不够导致的，再擦一遍没有用
;G1 X60 F25000 ;   g1 x60 f25000；g1 x60 f25000；G1 x60 f25000；

M1002 gcode_claim_action : 14M1002 gcode_claim_actionM1002 gcode_claim_action
G29.2 S1 ;打开ABL（自动调平），S0是关闭


;===== bed leveling ==================================, = = = = =基床整平  ==================================
;M1002 judge_flag g29_before_print_flag
;M622 J1
;
;    M1002 gcode_claim_action : 1
;    G29 A X{first_layer_print_min[0]} Y{first_layer_print_min[1]} I{first_layer_print_size[0]} J{first_layer_print_size[1]}
;    M400
;    M500 ; save cali data

;M623
;===== bed leveling end ================================
;=====对于有纹理的PEI板，在归巢时降低喷嘴，因为喷嘴接触到纹理的最顶部 ==
;curr_bed_type={curr_bed_type}
;{if curr_bed_type=="Textured PEI Plate"}
;G29.1 Z{-0.04} ; for Textured PEI Plate
;{endif}
;========关掉灯和挤压温度等 ============
;M1002 gcode_claim_action : 0M1002 gcode_claim_action: 0
;M106 S0 ; turn off fan； m106；关闭风扇
;M106 P2 S0 ; turn off big fan； m106 p2 0；关掉大风扇； m106 p2 0；关闭大风扇；m106 p2 0
;M106 P3 S0 ; turn off chamber fanm106 p3 50；关闭室内风机

;M190 S[bed_temperature_initial_layer_single] ;wait for bed tempM190 S[bed_temperature_initial_layer_single]；等待床的温度M190 S[bed_temperature_initial_layer_single]；M190 S[bed_temperature_initial_layer_single];；


;==结束gd代码=== date: 2024124 =====================
;==结束gcode=== date: 2024124 =====================

M400 ; wait for buffer to clearM400;等待缓冲区清空
M140 S0 ; turn off bedM140 50；关床M140 50；关掉床头m140 50
M106 S0 ; turn off fanM106；关闭风扇
M106 P2 S0 ; turn off remote part cooling fanM106 p2 0；关闭遥控部件冷却风扇
M106 P3 S0 ; turn off chamber cooling fanM106 p3 50；关闭室冷却风扇



G92 E0 ; zero the extruderG92 e0；将挤出机调零
G1 E-0.8 F1800 ; retractG1 e-0.8 f1800；收回G1 e-0.8 f1800；retract1 e-0.8 f1800G1 e-0.8 f1800；retract1 e-0.8 f1800G1 e-0.8 f1800；retract1 e-0.8 f1800; G1 e-0.8 f1800G1 e-0.8 f1800
G91
G1 X+5 Y+5 F24000   G1 x 5 y 5 f24000G1 X 5 Y 5 F24000 G1 X 5 Y 5 F24000G1 X 5 Y 5 F24000   G1 x 5 y 5 f24000G1 X 5 Y 5 F24000 G1 X 5 Y 5 F24000G1 X 5 Y 5 F24000   G1 x 5 y 5 f24000G1 X 5 Y 5 F24000 G1 X 5 Y 5 F24000G1 X 5 Y 5 F24000   G1 x 5 y 5 f24000G1 X 5 Y 5 F24000 G1 X 5 Y 5 F24000
G90
G1 Z{max_layer_z + 3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点zG1 Z{max_layer_z   3} F900 ; lower z a littleG1 Z{max_layer_z 3} F900；稍微降低一点z
G1 X80 Y254 F30000 ; 回坑，x80处门槛低
G1 Y266 F3000   G1 y266 f3000G1 Y266 F3000 G1 Y266 F3000

M400 ; wait all motion doneM400;等待所有动作完成
M17 S   M17年代
G92 E0
M17 Z0.4 ; lower z motor current to reduce impact if there is something in the bottomM17 z0.4；降低z电机电流，以减少冲击，如果有东西在底部
{if (max_layer_z + 35.0) < 250}{if (max_layer_z 35.0) < 250}
    G1 Z{max_layer_z +35.0}  F2000
    G1 Z{max_layer_z +34.0}
{else}
    G1 Z250   F2000 ;可以加E-7
    G1 Z248
{endif}


 M109 S{nozzle_temperature_initial_layer[initial_extruder]+40} ; 喷嘴比设定高40度，等待，使余料流净
G1 X64 Y266 F12000 
G4 P5000      ;等待5秒钟喷嘴余料流净滴落
G1 X84 Y266 F30 ;缓慢移动10s

G1 X100 F12000 ; wipe
G1 X86 F24000 ；堵住
;撞四氟管抖掉残料，本注释换行否则不执行还丢步

; pull back filament to AMS;将灯丝拉回AMS
M620 S255
G1 X20 Y50 F12000   G1 x20 y50 f12000
G1 Y-3
T255;抽回ams？
;G1 X80 F12000;这里给80似乎没有用，
;给80是想让他切完回坑不带渣渣，但ams抽完了才执行
G1 Y266;同上
G1 X70
M621 S255 ;抽回ams？


M106 P1 S255 ;打开吹料风扇 
M104 S0 
;关热端，如果在切料前执行会导致切料会等待降温
G1 X100 Y266 F12000   G1 x100 y266 f12000G1 X100 Y266 F12000 G1 X100 Y266 F12000
G1 X100 Y263 ;261就蹭螺丝上了
G1 X65 Y263   G1 x65 y263G1 X65 Y263 G1 X65 Y263
G1 X65 Y265   G1 x65 y265G1 X65 Y265 G1 X65 Y265
G1 X100 Y265 F1500   G1 x100 y265 f1500G1 X100 Y265 F1500 G1 X100 Y265 F1500
G1 X100 Y262   G1 x100 y262G1 X100 Y262 G1 X100 Y262
G1 X65 Y262   G1 x65 y262G1 X65 Y262 G1 X65 Y262
G1 X65 Y263   G1 x65 y263G1 X65 Y263 G1 X65 Y263
G1 X100 Y263 F30000   G1 x100 y263 f30000G1 X100 Y263 F30000 G1 X100 Y263 F30000
G1 X100 Y262   G1 x100 y262G1 X100 Y262 G1 X100 Y262
G1 X65 Y262   G1 x65 y262G1 X65 Y262 G1 X65 Y262
M106 P1 S0 ;关闭吹料风扇 

M622.1 S1 ; for prev firware, default turned onM622.1 s1；对于以前的固件，默认是打开的
M1002 judge_flag timelapse_record_flagM1002 judge_flag timeelapse_record_flagM1002 judge_flag timeelapse_record_flag
M622 J1   M622 j - 1
    M400 ; wait all motion doneM400;等待所有动作完成
    M991 S0 P-1 ;end smooth timelapse at safe posM991 S0 P-1；在安全位置结束平滑延时
    M400 S3 ;wait for last picture to be takenM400 S3；等待拍摄最后一张照片
M623; end of "timelapse_record_flag"M623;结束“timelapse_record_flag”M623;end of "timelapse_record_flag"M623；


M400 P100
M17 R ; restore z currentm17r；恢复z电流



M220 S100  ; Reset feedrate magnitudeM220 s100；重置馈电幅度
M201.2 K1.0 ; Reset acc magnitudeM201.2 k1.0；重置acc振幅
M73.2   R1.0 ;Reset left time magnitudeM73.2 R1.0；复位左时间幅度
M1002 set_gcode_claim_speed_level : 0


M620 S[next_extruder]A
M204 S9000
{if toolchange_count > 1 && (z_hop_types[current_extruder] == 0 || z_hop_types[current_extruder] == 3)}
G17
G2 Z{z_after_toolchange + 0.4} I0.86 J0.86 P1 F10000 ; spiral lift a little from second lift
{endif}
G1 Z{max_layer_z + 3.0} F1200

G1 X70 F25000
G1 Y245
G1 Y265 F15000
M400
M106 P1 S0
M106 P2 S0
{if old_filament_temp > 142 && next_extruder < 255}
M104 S[old_filament_temp]
{endif}
{if long_retractions_when_cut[previous_extruder]}
M620.11 S1 I[previous_extruder] E-{retraction_distances_when_cut[previous_extruder]} F{old_filament_e_feedrate}
{else}
M620.11 S0
{endif}
M400
G1 X90 F3000
G1 Y255 F4000
G1 X100 F5000
G1 X120 F15000
G1 X20 Y50 F21000
G1 Y-3
{if toolchange_count == 2}
; get travel path for change filament
M620.1 X[travel_point_1_x] Y[travel_point_1_y] F21000 P0
M620.1 X[travel_point_2_x] Y[travel_point_2_y] F21000 P1
M620.1 X[travel_point_3_x] Y[travel_point_3_y] F21000 P2
{endif}
M620.1 E F[old_filament_e_feedrate] T{nozzle_temperature_range_high[previous_extruder]}
T[next_extruder]
M620.1 E F[new_filament_e_feedrate] T{nozzle_temperature_range_high[next_extruder]}

{if next_extruder < 255}
{if long_retractions_when_cut[previous_extruder]}
M620.11 S1 I[previous_extruder] E{retraction_distances_when_cut[previous_extruder]} F{old_filament_e_feedrate}
M628 S1
G92 E0
G1 E{retraction_distances_when_cut[previous_extruder]} F[old_filament_e_feedrate]
M400
M629 S1
{else}
M620.11 S0
{endif}
G92 E0
{if flush_length_1 > 1}
M83
; FLUSH_START
; always use highest temperature to flush
M400
{if filament_type[next_extruder] == "PETG"}
M109 S260
{elsif filament_type[next_extruder] == "PVA"}
M109 S210
{else}
M109 S[nozzle_temperature_range_high]
{endif}
{if flush_length_1 > 23.7}
G1 E23.7 F{old_filament_e_feedrate} ; do not need pulsatile flushing for start part
G1 E{(flush_length_1 - 23.7) * 0.02} F50
G1 E{(flush_length_1 - 23.7) * 0.23} F{old_filament_e_feedrate}
G1 E{(flush_length_1 - 23.7) * 0.02} F50
G1 E{(flush_length_1 - 23.7) * 0.23} F{new_filament_e_feedrate}
G1 E{(flush_length_1 - 23.7) * 0.02} F50
G1 E{(flush_length_1 - 23.7) * 0.23} F{new_filament_e_feedrate}
G1 E{(flush_length_1 - 23.7) * 0.02} F50
G1 E{(flush_length_1 - 23.7) * 0.23} F{new_filament_e_feedrate}
{else}   其他{}
G1 E{flush_length_1} F{old_filament_e_feedrate}
{endif}
; FLUSH_END
G1 E-[old_retract_length_toolchange] F1800
G1 E[old_retract_length_toolchange] F300
{endif}

{if flush_length_2 > 1}

G91
G1 X3 F12000; move aside to extrude
G90
M83

; FLUSH_START
G1 E{flush_length_2 * 0.18} F{new_filament_e_feedrate}G1 E{flush_length_2 * 0.18} F{new_filament_feedrate}
G1 E{flush_length_2 * 0.02} F50
G1 E{flush_length_2 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_2 * 0.02} F50
G1 E{flush_length_2 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_2 * 0.02} F50
G1 E{flush_length_2 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_2 * 0.02} F50
G1 E{flush_length_2 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_2 * 0.02} F50
; FLUSH_END
G1 E-[new_retract_length_toolchange] F1800G1 E-[new_retract_length_toolchange
G1 E[new_retract_length_toolchange] F300G1 E[new_retract_length_toolchange
{endif}

{if flush_length_3 > 1}

G91
G1 X3 F12000; move aside to extrude
G90
M83   它

; FLUSH_START
G1 E{flush_length_3 * 0.18} F{new_filament_e_feedrate}G1 E{flush_length_3 * 0.18} F{new_filament_feedrate}
G1 E{flush_length_3 * 0.02} F50G1 E{flush_length_3 * 0.02
G1 E{flush_length_3 * 0.18} F{new_filament_e_feedrate}G1 E{flush_length_3 * 0.18} F{new_filament_feedrate}
G1 E{flush_length_3 * 0.02} F50G1 E{flush_length_3 * 0.02
G1 E{flush_length_3 * 0.18} F{new_filament_e_feedrate}G1 E{flush_length_3 * 0.18} F{new_filament_feedrate}
G1 E{flush_length_3 * 0.02} F50G1 E{flush_length_3 * 0.02
G1 E{flush_length_3 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_3 * 0.02} F50
G1 E{flush_length_3 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_3 * 0.02} F50
; FLUSH_END
G1 E-[new_retract_length_toolchange] F1800
G1 E[new_retract_length_toolchange] F300
{endif}

{if flush_length_4 > 1}

G91
G1 X3 F12000; move aside to extrude
G90
M83

; FLUSH_START
G1 E{flush_length_4 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_4 * 0.02} F50
G1 E{flush_length_4 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_4 * 0.02} F50
G1 E{flush_length_4 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_4 * 0.02} F50
G1 E{flush_length_4 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_4 * 0.02} F50
G1 E{flush_length_4 * 0.18} F{new_filament_e_feedrate}
G1 E{flush_length_4 * 0.02} F50
; FLUSH_END
{endif}
; FLUSH_START
M400
M109 S[new_filament_temp]
G1 E2 F{new_filament_e_feedrate} ;Compensate for filament spillage during waiting temperature
; FLUSH_END
M400
G92 E0
G1 E-[new_retract_length_toolchange] F1800
M106 P1 S255
M400 S3

G1 X70 F5000
G1 X90 F3000
G1 Y255 F4000
G1 X105 F5000
G1 Y265 F5000
G1 X70 F10000
G1 X100 F5000
G1 X70 F10000
G1 X100 F5000

G1 X70 F10000   G1 x70 f10000
G1 X80 F15000
G1 X60
G1 X80
G1 X60
G1 X80 ; shake to put down garbageG1 x80；摇一摇放下垃圾
G1 X100 F5000   G1 x100 f5000
G1 X165 F15000; wipe and shakeG1 x165 f15000；擦干并摇动
G1 Y256 ; move Y to aside, prevent collisionG1 y256；将Y移到一边，防止碰撞
M400
G1 Z{max_layer_z + 3.0} F3000G1 Z{max_layer_z 3.0} F3000
{if layer_z <= (initial_layer_print_height + 0.001)}{如果layer_z <= (initial_layer_print_height 0.001)}
M204 S[initial_layer_acceleration]M204年代(initial_layer_acceleration)
{else}   其他{}
M204 S[default_acceleration]M204年代(default_acceleration)
{endif}
{else}   其他{}
G1 X[x_after_toolchange] Y[y_after_toolchange] Z[z_after_toolchange] F12000
{endif}
M621 S[next_extruder]A

