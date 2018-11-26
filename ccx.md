## Metodología

De la población de pacientes en postoperatorio, intervenidos por cirugía gastrointestinal en un centro asistencial de alta complejidad en la ciudad de Pereira, se toma una muestra aleatoria n=22, de quienes se registra la edad, el sexo, el diagnóstico, el tipo de procedimiento, si se realizó abordaje laparoscópico, la presencia de complicaciones, el grado de severidad de la complicación y la necesidad de reintervención en los 5 días posteriores a cirugía, con la meta de comparar estas características con las diferencias entre la medición del valor de la PCR y los valores de hemograma en el dia 3 con respecto al día 5 de cada caso, para estimar el comportamiento de la media y la varianza, teniendo como objetivo proponer un modelo matemático que relacione un valor de referencia estándar para la PCR y las variables del hemograma que muestran un patrón de dependencia con la presencia de complicaciones, con sus grados de severidad y la necesidad de reintervención. 

Usando Jupyter Notebook con el kernel de R, se describe las características de la muestra y se identifica la variable del hemograma que se ajuste linealmente a la PCR; en esta muestra de pacientes, el CAL y el CAN son colineales y muestran un patrón de variación directamente proporcional con la PCR en la mayoría de los casos. 

La tabla general contienen los datos registrados, la tabla descriptiva presenta el sumario estadístico de las variables continuas, posteriormente se presentan los gráficos de dispersión comparativos, con las líneas de regresión y distribuciones explícitas de cada variable, donde se sustenta el sentido de la relación entre la PCR y el CAL.


```R
# Librerías y paquetes utilizadas
install.packages("RcmdrMisc")
install.packages("googlesheets")
install.packages("dplyr")
install.packages("FSA")
library("googlesheets")
library("FSA")
library("psych")
library("gmodels")
library("car")
suppressMessages(library(dplyr))
```


```R
# Conexión a las hojas de calculo de Google
my_sheets <- gs_ls()
gs_ls("CCX")
gap <- gs_title("CCX")
gs_ws_ls(gap)
```


```R
# Tabla General
gap %>% gs_read(ws = 1)
```


```R
CCXtg <- gap %>% gs_read(ws = "Tabla General")
CCXtca <- gap %>% gs_read(ws = "Tabla de Categorías")
```


```R
describe(CCXtg[c(9:22)],
         type=1)
```


<table>
<thead><tr><th></th><th scope=col>vars</th><th scope=col>n</th><th scope=col>mean</th><th scope=col>sd</th><th scope=col>median</th><th scope=col>trimmed</th><th scope=col>mad</th><th scope=col>min</th><th scope=col>max</th><th scope=col>range</th><th scope=col>skew</th><th scope=col>kurtosis</th><th scope=col>se</th></tr></thead>
<tbody>
	<tr><th scope=row>PCRD3</th><td> 1         </td><td>22         </td><td>181.036364 </td><td>90.3793806 </td><td>151.255    </td><td>175.919444 </td><td>80.082639  </td><td>  43.71    </td><td>396.21     </td><td>352.50     </td><td> 0.57975356</td><td>-0.2871839 </td><td>19.2689487 </td></tr>
	<tr><th scope=row>LeuD3</th><td> 2         </td><td>22         </td><td> 12.445000 </td><td> 4.7735649 </td><td> 12.050    </td><td> 12.434444 </td><td> 4.247649  </td><td>   2.41    </td><td> 22.40     </td><td> 19.99     </td><td> 0.03119877</td><td>-0.1703611 </td><td> 1.0177274 </td></tr>
	<tr><th scope=row>NeutD3</th><td> 3         </td><td>22         </td><td> 10.119545 </td><td> 4.5996185 </td><td>  9.790    </td><td>  9.903889 </td><td> 4.069737  </td><td>   1.86    </td><td> 20.70     </td><td> 18.84     </td><td> 0.39158756</td><td>-0.2116967 </td><td> 0.9806420 </td></tr>
	<tr><th scope=row>LinfD3</th><td> 4         </td><td>22         </td><td>  1.340909 </td><td> 0.6684090 </td><td>  1.460    </td><td>  1.378333 </td><td> 0.882147  </td><td>   0.08    </td><td>  2.24     </td><td>  2.16     </td><td>-0.43015876</td><td>-0.9750922 </td><td> 0.1425053 </td></tr>
	<tr><th scope=row>VCMD3</th><td> 5         </td><td>22         </td><td> 84.600000 </td><td> 7.6410545 </td><td> 86.750    </td><td> 85.072222 </td><td> 8.376690  </td><td>  69.20    </td><td> 96.20     </td><td> 27.00     </td><td>-0.47616947</td><td>-0.8245584 </td><td> 1.6290783 </td></tr>
	<tr><th scope=row>HbD3</th><td> 6         </td><td>22         </td><td> 10.622273 </td><td> 1.5235514 </td><td> 10.350    </td><td> 10.566667 </td><td> 1.097124  </td><td>   7.78    </td><td> 13.90     </td><td>  6.12     </td><td> 0.42778186</td><td>-0.2148599 </td><td> 0.3248223 </td></tr>
	<tr><th scope=row>PCRD5</th><td> 7         </td><td>22         </td><td>127.382727 </td><td>87.6192666 </td><td> 88.240    </td><td>119.364444 </td><td>95.909394  </td><td>  22.88    </td><td>319.50     </td><td>296.62     </td><td> 0.63309540</td><td>-0.6721781 </td><td>18.6804904 </td></tr>
	<tr><th scope=row>LeuD5</th><td> 8         </td><td>22         </td><td> 10.005909 </td><td> 3.8202314 </td><td>  8.985    </td><td>  9.411111 </td><td> 2.802114  </td><td>   5.48    </td><td> 20.10     </td><td> 14.62     </td><td> 1.39583152</td><td> 1.3524529 </td><td> 0.8144761 </td></tr>
	<tr><th scope=row>NeutD5</th><td> 9         </td><td>22         </td><td>  7.557727 </td><td> 3.9040625 </td><td>  6.475    </td><td>  6.856667 </td><td> 2.527833  </td><td>   3.49    </td><td> 18.10     </td><td> 14.61     </td><td> 1.59065862</td><td> 1.9110300 </td><td> 0.8323489 </td></tr>
	<tr><th scope=row>LinfD5</th><td>10         </td><td>22         </td><td>  1.308636 </td><td> 0.6861091 </td><td>  1.245    </td><td>  1.317222 </td><td> 0.904386  </td><td>   0.10    </td><td>  2.36     </td><td>  2.26     </td><td>-0.04197772</td><td>-1.1238819 </td><td> 0.1462789 </td></tr>
	<tr><th scope=row>VCMD5</th><td>11         </td><td>22         </td><td> 83.936364 </td><td> 7.5264325 </td><td> 85.850    </td><td> 84.005556 </td><td> 8.228430  </td><td>  71.00    </td><td> 97.70     </td><td> 26.70     </td><td>-0.14542889</td><td>-0.9634291 </td><td> 1.6046408 </td></tr>
	<tr><th scope=row>HbD5</th><td>12         </td><td>22         </td><td> 10.562727 </td><td> 1.4814317 </td><td> 10.350    </td><td> 10.556111 </td><td> 1.519665  </td><td>   8.11    </td><td> 13.60     </td><td>  5.49     </td><td> 0.07554940</td><td>-0.6483924 </td><td> 0.3158423 </td></tr>
	<tr><th scope=row>DiPCR</th><td>13         </td><td>22         </td><td>-53.653636 </td><td>64.4227873 </td><td>-59.535    </td><td>-60.669444 </td><td>49.578144  </td><td>-144.00    </td><td>136.26     </td><td>280.26     </td><td> 1.11012519</td><td> 1.7467873 </td><td>13.7349844 </td></tr>
	<tr><th scope=row>DiLeu</th><td>14         </td><td>22         </td><td> -2.439091 </td><td> 2.7411049 </td><td> -3.005    </td><td> -2.406111 </td><td> 0.719061  </td><td> -10.26    </td><td>  3.64     </td><td> 13.90     </td><td>-0.35244720</td><td> 1.9997223 </td><td> 0.5844055 </td></tr>
</tbody>
</table>




```R
boxplot(CCXtg$PCRD3~CCXtg$CCX,
        main="PCR día 3 - Complicaciones")
boxplot(CCXtg$PCRD3~CCXtg$GCCX,
        main="PCR día 3 - Grado de Complicación")
boxplot(CCXtg$PCRD3~CCXtg$RCX,
        main="PCR día 3 - Reintervención")
boxplot(CCXtg$PCRD5~CCXtg$CCX,
        main="PCR día 5 - Complicaciones")
boxplot(CCXtg$PCRD5~CCXtg$GCCX,
        main="PCR día 5 - Grado de Complicación")
boxplot(CCXtg$PCRD5~CCXtg$RCX,
        main="PCR día 5 - Reintervención")
boxplot(CCXtg$LeuD3~CCXtg$CCX,
        main="CAL día 3 - Complicaciones")
boxplot(CCXtg$LeuD3~CCXtg$GCCX,
        main="CAL día 3 - Grado de Complicación")
boxplot(CCXtg$LeuD3~CCXtg$RCX,
        main="CAL día 3 - Reintervención")
boxplot(CCXtg$LeuD5~CCXtg$CCX,
        main="CAL día 5 - Complicaciones")
boxplot(CCXtg$LeuD5~CCXtg$GCCX,
        main="CAL día 5 - Grado de Complicación")
boxplot(CCXtg$LeuD5~CCXtg$RCX,
        main="CAL día 5 - Reintervención")
boxplot(CCXtg$DiPCR~CCXtg$CCX,
        main="Diferencia de la PCR - Complicaciones")
boxplot(CCXtg$DiLeu~CCXtg$CCX,
        main="Diferencia del CAL- Complicaciones")
boxplot(CCXtg$DiPCR~CCXtg$GCCX,
        main="Diferencia de la PCR - Grado de Complicación")
boxplot(CCXtg$DiLeu~CCXtg$GCCX,
        main="Diferencia del CAL- Grado de Complicación")
boxplot(CCXtg$DiPCR~CCXtg$RCX,
        main="Diferencia de la PCR - Reintervención")
boxplot(CCXtg$DiLeu~CCXtg$RCX,
        main="Diferencia del CAL- Reintervención")
```


```R
# Gráfico de dispersión de todas las variables continuas
scatterplotMatrix(CCXtg[c(9:22)])
```


```R
scatterplotMatrix(CCXtg[c(9,15,21,10,16,22)])
```


```R
scatterplotMatrix(CCXtg[c(9,15,21)])
```


```R
scatterplotMatrix(CCXtg[c(10,16,22)])
```


```R
CCXtc <- gap %>% gs_read(ws = "Tabla de Cálculos")
```


```R
describe(CCXtc[c(1:4,7:13,15)],
         type=1)
```


```R
# Gráfico de dispersión de todas las variables continuas
scatterplotMatrix(CCXtc[c(1:6,9:15,17)])
```


```R
boxplot(CCXtc$Cr~CCXtc$CCX,
        main="Coeficiente de Relación y Complicaciones")
boxplot(CCXtc$Cr~CCXtc$GCCX,
        main="Coeficiente de Relación y Grado de Complicaciones")
boxplot(CCXtc$Cr~CCXtc$RCX,
        main="Coeficiente de Relación y necesidad de reintervención")
boxplot(CCXtc$AaXCr~CCXtc$CCX,
        main="Area x Coeficiente de Relación y Complicaciones")
boxplot(CCXtc$AaXCr~CCXtc$GCCX,
        main="Area x Coeficiente de Relación y Grado de Complicaciones")
boxplot(CCXtc$AaXCr~CCXtc$RCX,
        main="Area x Coeficiente de Relación y necesidad de reintervención")
```


```R
CrossTable(CCXtc$CCX, CCXtc$pCCX)
CrossTable(CCXtc$CCX, CCXtc$Cr)
CrossTable(CCXtc$GCCX, CCXtc$pCCX)
CrossTable(CCXtc$GCCX, CCXtc$Cr)
```


```R
attach(CCXtc)
tablapCCX <- table(CCXtc$CCX,CCXtc$pCCX)
tablaCr <- table(CCXtc$CCX,CCXtc$Cr)
tablapCCX2 <- table(CCXtc$GCCX,CCXtc$pCCX)
tablaCr2 <- table(CCXtc$GCCX,CCXtc$Cr)
tablapCCX3 <- table(CCXtc$RCX,CCXtc$pCCX)
tablaCr3 <- table(CCXtc$RCX,CCXtc$Cr)
```


```R
tablapCCX <- xtabs(~CCX+pCCX)
summary(tablapCCX)
tablapCCX <- xtabs(~CCX+Cr)
summary(tablaCr)
tablapCCX2 <- xtabs(~GCCX+pCCX)
summary(tablapCCX2)
tablapCCX2 <- xtabs(~GCCX+Cr)
summary(tablaCr2)
tablapCCX3 <- xtabs(~RCX+pCCX)
summary(tablapCCX3)
tablapCCX3 <- xtabs(~RCX+Cr)
summary(tablaCr3)
```
