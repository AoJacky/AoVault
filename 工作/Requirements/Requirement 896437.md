[Requirement 896437](https://dev.azure.com/ABB-MO-Drives/CN%20Servo%20SW/_workitems/edit/896437): [E 550/E 530]Encoder automatic recognition

### 需求描述

### 接收标准

### 更新后的识别流程

``` mermaid
flowchart TD
a0((Default))-->a1(ReStart)
a1-->a2("P17.03 == 5 \n Default Value \n 23位光编电子多圈")
a2-->a3{P17.00 == ?}
a3-->|==0| a4(Build-in)
a3-->|==1| a5(User input)
a3-->|==2| a6(Encoder)
a4-->a7(P17.03 = User Set)
a5-->a7
a6-->a8{Read MTP TypeCode}
a8-->|else| a11(Report F0105)
a8-->|==3| a9{Read EncoderID}
a8--->|==4| a10(P17.03 == 6 \n 23位光编机械多圈)
a9-->|else| a11
a9-->|==06H| a12(P17.03 == 4 \n 23位光编电子单圈)
a9-->|==17H| a13(P17.03 == 5 \n 23位光编电子多圈)

```
