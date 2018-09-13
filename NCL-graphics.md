> This is my NCL graphics notes

### 1  Resource  Reference
#### 1.1 workstation resource
1. png
```NCL
wks  = gsn_open_wks("png","fig")
system("convert fig.png -trim +repage fig.png")   ; trim white margin
```

#### 1.2 gsn resource
```NCL
res@gsnMaximize        = True  ; plot drawn will be maximized in the workstation
res@gsnAddCyclic       = False ; if plot global data:True; else : False
res@gsnDraw            = False ; not draw first
res@gsnFrame           = False ; not page turning
res@gsnMajorLatSpacing = 15    ; set lat tickmark spacing. if plot profile,use gsn_csm_hov()
res@gsnMajorLonSpacing = 15    ; set lon tickmark spacing
```
#### 1.3 map resource
```NCL
res@mpOutlineBoundarySets = "National" ; boundary outlines. case:"NoBoundaries"
```
#### 1.4 tickmark resource
#### 1.5 contour resource

#### 1.6 vector resource
1. WindBarb
```NCL
res@vcGlyphStyle               = "WindBarb"
res@vcWindBarbLineThicknessF   = 0.8
res@vcWindBarbColor            = "black"
res@vcRefLengthF               = 0.02 ;0.045
res@vcRefMagnitudeF            = 10   ; uv need multiply by 2.5
res@vcRefAnnoOn                = True ; 
res@vcRefAnnoString1           = "4 m/s"
res@vcRefAnnoSide              = "Top"
res@vcRefAnnoString2On         = False
res@vcRefAnnoPerimOn           = False
res@vcRefAnnoOrthogonalPosF    = -0.09 ; Bottom is -1, Top is 0
res@vcRefAnnoParallelPosF      = 0.99;0.999  Leftmost is 0, Rightmost is 1
res@vcRefAnnoBackgroundColor   = "gray90"
res@vcMinDistanceF             = 0.02
res@vcMapDirection             = False ; Sectional view need set False
```



#### 1.7 label bar resource
```NCL
res@lbOrientation                   = "Vertical"
res@pmLabelBarWidthF                = 0.05
res@pmLabelBarOrthogonalPosF        = -1.07   ; Left is -1, right is 0
```

### 2 Examples

#### 2.1 mask China map
> China map supported by 
