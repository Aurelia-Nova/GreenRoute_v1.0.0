# GreenRoute_v1.0.0

## 1. 安装与首次启动

建议使用 Windows 10/11 x64 和 Python 3.14；兼容目标为 Python 3.11–3.14。压缩包不包含 Python 解释器或完整依赖，不是免安装的 EXE。

1. 将整个文件夹解压到可写目录，例如 D:\ResearchTools\GreenRoute。不要只复制 app.py。
2. 双击 start_windows.bat。首次会创建独立 .venv 并联网安装依赖。启动器会优先寻找 Python 3.14，找不到时尝试系统 Python。
3. 浏览器打开 http://localhost:8501。首次自行创建管理员账号，密码至少 12 字符，没有默认密码。
4. 管理员可在“账户与安全”中创建其它化学家账号。普通用户不能开放注册其它账号。
5. 依赖安装完成后，本地路线规划、库存、账号和文件操作可离线运行。在线资料查询和远程 AI 仍需要对应网络。
6. 关闭软件时在启动窗口按 Ctrl+C。项目保存在软件目录 var/greenroute.sqlite3；升级前备份整个 var 目录，不要只拷贝 .sqlite3 而遗漏正在使用的 WAL 文件。

如果出现 no module named pip，启动器会尝试 ensurepip 修复。若依赖下载失败，应由 IT 组检查 Python 包源、代理、证书或预先制作 wheelhouse，不要禁用证书验证。

Linux/macOS：

```bash
chmod +x start_linux_mac.sh
./start_linux_mac.sh
```

手动安装：

```bash
python -m venv .venv
# Windows 使用 .venv\Scripts\python.exe；Linux/macOS 使用 .venv/bin/python
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

手动命令中的 python 必须指向新建虚拟环境，而不是未激活环境的其它解释器。默认仅监听本机。不要将这个版本直接暴露到公网。



## 2. 路线设计与可信度

先用乙酸苯胺、联苯、苯甲酸乙酯验证工作流，再使用已完成的项目回放。目标支持 SMILES、InChI 和 MOL/SDF 首个记录。批量模式接受最多五个 SMILES。最大反应步数是总步数预算，不是最长线性序列。搜索有预算限制，因此不是全空间最优性证明。

内置目录是演示数据，出现“可得”不代表真实有货。上传的库存以规范化结构精确覆盖演示条目。关闭演示目录可只按本账户库存搜索。盐型、溶剂化物、互变异构体和立体异构体没有做等价映射。默认不把未查证的小分子直接当作可采购原料。

每步可信度按规则先验和选择性风险计算，并对立体控制问题作扣减。

路线级可信度为单步指数的几何平均值，作为相对质量指标，不将各步数值相乘冒充整条路线成功率。综合分主要考虑可操作性和规则可信度，再叠加原料可得、安全和绿色属性，额外惩罚增加的步骤。所有指标均可以在报告中追溯。

本版没有按数据集校准概率、置信区间或湿实验验证。实验反馈会保存，但不会未经审核自动提高规则可信度。

## 3. 资料、图谱与计算

“化合物资料”支持从路线中选择目标、中间体和起始物。联网前需要确认检索标识符和发送授权。名称/CAS检索可能有多个CID，必须选择记录，不能将不同盐型或异构体的数据合并为同一化合物。

PubChem PUG REST 读取结构和计算/汇总属性，PUG View 读取实测性质、光谱及安全方面的来源注释。返回结果保存来源、引用编号和获取时间。某项没有数据时，应视为缺失，不能由AI补造。标准图谱入口可能是文献或数据库外链，而不是可直接下载的完整原始谱图，应查看实验/预测属性、溶剂、频率、样品状态和原始来源。

SDS/MSDS不等同于危险性注释。正式使用时应保存对应供应商、产品编号、浓度、语言及修订日期的SDS。文件可作为化合物附件保存，软件不会将未经审核的附件自动认定为正确。

本地3D采用ETKDG构象生成以及可用的MMFF94/UFF优化，只生成一个构象，不保证全局最低能量。力场能量和Gasteiger电荷不是DFT能量、电荷、HOMO/LUMO或实验性质。真正量子化学结果以QM证据JSON导入，必须记录方法、基组、单位和来源，ORCA输入导出只准备文件，不执行计算。

## 4. AI、数字生命与LLM

API密钥不填写进代码、配置JSON或网址。先在界面查看本账户变量前缀，再由IT组在启动进程环境中设置 GREENROUTE_KEY_<账户前缀>_<名称>。软件只允许当前账户引用自己的前缀变量。内网或本地接口需管理员额外授权，详见 docs/INTEGRATIONS.md。

SKILL.md或ZIP导入到当前账户后，可在下拉菜单切换。ZIP只读取文本、限制解压大小并拒绝路径穿越；脚本不运行，不继承工具权限。它不是远程代码插件系统。模型不能凭skill取得实验设备、文件系统、其它账户或网络工具的执行权限。

发送到外部接口前，每条对话都需要发送授权。是否附带路线是单独选项。skill和已选上下文会包含在提示词中；如果其中有保密信息，应优先使用基地自部署模型。账号密码属于本软件，不是第三方服务账号的通用登录凭证。

## 5. 互联、维护与测试

软件互联详见 docs/INTEGRATIONS.md，安全与部署见 docs/SECURITY.md。在界面创建带权限、有效期和可撤销的令牌后，另行运行 start_api_windows.bat 或 python api_server.py。

```bash
python -m pip install -r requirements-dev.txt
python -m pytest
python cli.py backup D:/Backups/greenroute_2026-10-26.sqlite3
```

备份命令拒绝覆盖已存在文件。备份含研究资料、口令哈希和令牌哈希，应按敏感数据管理。若通过直接复制目录备份，请先关闭软件，在线备份优先使用CLI的SQLite备份接口。

## 6. 官方接口依据

- PubChem PUG REST：https://pubchem.ncbi.nlm.nih.gov/docs/pug-rest
- PubChem PUG View：https://pubchem.ncbi.nlm.nih.gov/pug_view/
- NIST Chemistry WebBook：https://webbook.nist.gov/
- AIST SDBS：https://sdbs.db.aist.go.jp/
- Elsevier数据访问条款与API说明：https://dev.elsevier.com/
- RDKit：https://www.rdkit.org/docs/

## 7. 关于更新

- 软件定位是辅助人类化学家设计合成路线，同时兼顾可行性和绿色化学。
- 软件的智能性高度依赖数据集，仅内测版可接入基地数据库，还是本猫。
- 原则上，访问商业数据库需要合法账密。
- 在后续的迭代中，会逐步接入全自动分析化学工作台、大语言模型和用户的skills。

