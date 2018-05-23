> This is my NCL graphics notes

### 1  Resource  Reference
#### 1.1 workstation resource
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
#### 1.7 label bar resource

### 2 Examples

#### 2.1 mask China map
> China map supported by 
