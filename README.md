# bambu-p1-jilv-gcode


<h1 style   风格   风格="font-size: 40px;">急律gcode</h1>



## 项目介绍：

拓竹p1的急律gcode，不磨嘴、无清理线、精简步骤、快速开打、0维护.已经过半年连续测试

需要自己调试zoffset，适合有打印机调平经验的玩家。不良反应：不可避免的概率性没擦干净嘴(概率和官方差不多)

建议观看视频BV1ddS2YREu6

## 使用注意


;原理是打完再清理喷嘴，节省掉下次开打清理的时间，所以首次运行前擦干净喷嘴！！

;首次用擦不干净喷嘴为正常现象，循环两三次就好

;归零点在床面右下角，一般会比床中心高，所以需要补偿zoffset（参考值-0.28，拓竹切片没有，用orca）

——不会调的也给个-0.25一般也能使（？

;开始和结束g代码都在同一个文件里，分别复制进切片的开始和结束gcode

;如果你是从别人手上拿到本文件，强烈建议看一遍b站视频：BV1ddS2YREu6

;交流群：522185942，欢迎反馈问题、合作开发~

### 开发者注意！
注释过多会使g代码不执行，注释请用英文分号
