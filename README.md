# read_CN-H1 - README

## 简介

`read_h1.py` 是一个用于读取 MGrid 数据并计算和绘制磁场分布的 Python 脚本。该脚本使用了 `numpy`、`scipy.io` 和 `matplotlib` 等库，并且需要安装 `mgrid`（from SIMSOPT） 包。

## 依赖

在运行此脚本之前，请确保安装了以下 Python 包：

```bash
pip install numpy scipy matplotlib
```

## 使用说明

1. **设置电流**

   在脚本中，您可以设置线圈的电流值。脚本中定义了不同类型的线圈：

   - TF: Toroidal Field Coils（环向场线圈）
   - PF: Poloidal Field Coil（极向场线圈）
   - HCW: Helical winding Coil（螺旋绕组线圈）
   - IVF: Inner Vertical Field（内垂直场）
   - OVF: Outer Vertical Field（外垂直场）

   您可以通过修改以下变量来设置电流：

   ```python
   current = 5000 # PPS（脉冲电源）电流范围：1.5e3 < current < 1.4e4 (A)，MG电流范围：current < 1500 (A)

   Is_raw = False
   if Is_raw:
       TF_current = 1.0 
       PF_current = 1.0
       HCW_current = 1.0
       IVF_current = 1.0
       OVF_current = 1.0
   else:
       TF_current = current * 10
       PF_current = current
       HCW_current = current * kappa_h
       IVF_current = -current * 16
       OVF_current = -current * 8 * kappa_v
   ```

2. **读取 MGrid 文件**

   脚本默认读取文件 `mgrid_h1s.nc`，并提取线圈名称和网格信息：

   ```python
   nc_file = "mgrid_h1s.nc"
   mgrid_data = mgrid.MGrid.from_file(nc_file)
   coil_list = mgrid_data.coil_names
   r_max, r_min, z_max, z_min = mgrid_data.rmax, mgrid_data.rmin, mgrid_data.zmax, mgrid_data.zmin
   ```

3. **计算磁场**

   根据设置的电流，计算实际的磁场分布：

   ```python
   currents_group = [TF_current, PF_current, HCW_current, IVF_current, OVF_current]
   Bp, Br, Bz = 0, 0, 0
   for i, current in enumerate(currents_group):
       Bp += mgrid_data.bp_arr[i] * current
       Br += mgrid_data.br_arr[i] * current
       Bz += mgrid_data.bz_arr[i] * current

   B = np.sqrt(Bp**2 + Br**2 + Bz**2)
   ```

4. **绘制磁场**

   脚本使用 `matplotlib` 绘制磁场分布图：

   ```python
   import matplotlib.pyplot as plt
   fig, ax = plt.subplots(1, 1, figsize=(8, 8))
   ax.set_aspect('equal')
   ax.set_xlim(r_min, r_max)
   ax.set_ylim(z_min, z_max)
   ax.set_xlabel('R (m)')
   ax.set_ylabel('Z (m)')
   ax.set_title('|Bp| (T) at \phi = {:.0f}'.format(np.rad2deg(phi_range[phi_index])))
   B_contourf = plt.colorbar(ax.contourf(r_grid * grid_unit, z_grid * grid_unit, B_plotted_clipped, 100, cmap='jet'))
   if phi_angle == 0:
       ax.plot(Plasma_center, Z_center, 'ro')
   plt.show()
   ```

## 注意事项

- 请确保 MGrid 数据文件 `mgrid_h1s.nc` 存在于脚本所在目录中。
- 根据需要修改脚本中的电流值和其他参数以适应您的实验需求。

## 许可证

此项目遵循 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

---

如果您有任何问题或建议，请通过 GitHub 提交 issue。
