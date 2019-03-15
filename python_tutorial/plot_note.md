#### colorbar设置
colorbar两边都延伸，设置为三角形
```python
colors  = np.array( [ [0,228,0], [255,255,0], [255,126,0], [255,0,0], [153,0,76], [126,0,35] ], dtype='f4')
levels  = np.array([50,100,150,200,250], dtype='f')
cmap , norm = mpl.colors.from_levels_and_colors(levels, colors/255,extend='both' )
im = ax.contourf(olon, olat, zvalue_new, levels=levels, extend='both' ,cmap=cmap, norm=norm, \
    transform=ccrs.PlateCarree()) # extend='both' ,
clb = plt.colorbar(im, extendfrac='auto',fraction=0.025, pad=0.01, )
clb.ax.tick_params(axis='y', length=0., width=0.3,direction='in',labelsize=8)
clb = plt.colorbar(im,  cax=inset_axes(ax, width="2.8%", height="68%", loc=3),extend='both',\ 
    extendfrac='auto', ) # extendrect=True,
clb.ax.tick_params(axis='y', length=0., width=0.3,direction='in',labelsize=6)  
```
<div align=center><img src="https://github.com/wangrenz/notes/blob/master/python_tutorial/figures/pm25_tri.png" width="70%" /></div>

设置为矩形
```python
clb = plt.colorbar(im,  cax=inset_axes(ax, width="2.8%", height="68%", loc=3),extend='both',\ 
    extendrect=True, extendfrac='auto', ) # extendrect=True,
```

![image](https://github.com/wangrenz/notes/blob/master/python_tutorial/figures/pm25_square.png)
