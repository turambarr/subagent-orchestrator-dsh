# NTN 定位仿真平台 · 领域附录

本附录给 flash 提供"零上下文也能干活"的环境事实与基准数字。派工 prompt 里照抄即可。

## 环境

- ssh 别名：`byubuntu`（远端，10.29.205.238）
- 项目根：`P=$(ls -d /home/bupt/*/ChatGPT/*定位)`
- 唯一解释器：`$P/envs/tudat/bin/python`（numpy 2.4.6 + tudatpy 1.1.0；系统 python3 无 numpy）
- 工作目录：`$P/implementation`

## 关键目录

| 路径 | 内容 |
|---|---|
| `$P/implementation/platform/scenes/` | 场景 JSON（iss-*.json、*.tle） |
| `$P/implementation/platform/` | scene.py / passive.py / run_scene.py |
| `$P/implementation/platform/results/` | 场景编译产物 |
| `$P/results/runs/` | OAI 联合运行（gnb/ue 日志、csi、capture iq） |
| `$P/openairinterface5g/radio/rfsimulator/` | physics.hpp / oai_adapter.hpp / simulator.cpp |
| `~/ntnwork/` | 校验脚本（csi_check2.py、toa3.py、sweep.py、passfind.py） |

## 运行命令

```bash
# 真值生成器（被动定位事件一致模型）
$P/envs/tudat/bin/python $P/implementation/platform/passive.py <scene> --out <dir>

# 物理 CSV + OAI 波形联合运行
$P/envs/tudat/bin/python $P/implementation/platform/run_scene.py \
  --scene <scene> --out <dir> --seconds <N>

# 改 C++ 信道后必须重建共享库
cd $P/openairinterface5g/build && ninja rfsimulator
```

## 数据契约（逐列）

- `dl.csv`：`t_s,tau_s,tau_dot,gain_db,doppler_hz,fspl_db,elevation_deg,rx_dbm,snr_db,same_epoch_delay_s,visible,fc_hz`
- `csi.csv`：`frame,slot,symbol,fft_size,rb_start,rb_count,bwp_start,first_carrier_offset,noise_var,freq_offset_hz,common_freq_offset_hz`
- `passive.csv`：24 列，含 `d_s,o_s,u_s,v_s,z_s,eps_corr_s,u_ref_s,resid_pred_m,reaction_protocol_s,reaction_device_s,reaction_net_s,theta_jitter_s`
- capture iq 块：`<u8 stamp><u4 nn><nn×complex64>`；source 块 `<u8 begin><u4 nn><nn×complex64>`

## 基准数字（一致性校验目标值）

| 校验 | 命令 | 基准 |
|---|---|---|
| CSI 多普勒 | `csi_check2.py <run_dir>` | corr 0.999988、MAD 10.6 Hz、median −27.5 Hz、p95 200.5 Hz、683 条 |
| 盲 TOA | `toa3.py <run_dir> <dl.csv>` | median −31.3 ns、p95 94.6 ns、max 110.5 ns（1 样本=130 ns） |
| passive 闭合（理想钟） | — | closure 42 µm |
| passive 闭合（0.2 ppm 漂移） | — | closure 0.84 m |

## 红线（flash 必须遵守）

1. `run_scene.py --seconds` 默认 12（只跑 9~12 s 场景），全过境必须显式传 `--seconds`。
2. 位置键名必须是 `height_m`（`alt_m` 会被静默忽略）。
3. 改 C++（physics.hpp / oai_adapter.hpp / simulator.cpp）必须 `ninja rfsimulator` 重建。
4. 观测量不得被 OAI NTN 分支直灌（`apply_ntn_timing_advance_and_doppler`，条件 `cell_specific_k_offset==0` 早退时才是自估）。
5. EOP 覆盖约束：场景 epoch 必须在 `~/.tudat/resource/earth_orientation/eopc04_14_IAU2000.62-now.txt` 覆盖范围内（已外推到 2026-12-31）。
