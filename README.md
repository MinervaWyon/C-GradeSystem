# 成绩管理系统 (Grade Management System)

C++ 课程大作业 — 一个基于 Qt5 的学生成绩管理系统，提供控制台和 GUI 两种交互方式。

## 团队成员

| 姓名 | 分工 |
|------|------|
| 吴彦彦 | 用户认证、数据持久化、性能测试 |
| 韦莹 | 学生档案管理、成绩管理 |
| 付初琰 | 统计排序、报表生成、性能测试修改 |
| 马文晨 | Qt GUI 界面设计与开发 |

## 目录结构

```
c++项目7.13/
├── src/                          # 源代码
│   ├── main.cpp                  # 控制台版主程序（完整功能）
│   ├── mainwindow.cpp / .h       # Qt GUI 主窗口
│   ├── qt_main.cpp               # Qt 程序入口
│   ├── GradeSystem.cpp / .h      # 成绩管理系统核心类
│   ├── Student.cpp / .h          # 学生类
│   ├── Grade.cpp / .h            # 成绩类
│   ├── FileManager.cpp / .h      # 文件读写管理
│   └── QtGradeSystem.pro         # Qt 项目文件
├── data/                         # 示例数据
│   ├── students.txt              # 学生数据（8人）
│   └── grades.txt                # 成绩数据（24条）
├── bin/                          # 可执行文件
│   └── QtGradeSystem.exe         # GUI 版（Release 构建）
├── docs/                         # 文档
│   ├── performance_report.txt    # 性能测试报告
│   └── perf_test.cpp             # 性能测试源码
├── 打包程序/                     # 完整发布包（含 Qt 运行时 DLL）
│   ├── QtGradeSystem.exe
│   ├── data/                     # 数据文件副本
│   └── *.dll                     # Qt5 及 MinGW 运行时依赖
└── README.md
```

## 功能列表

### 1. 用户认证
- 教师登录（默认账号：`admin` / `admin123`）
- 最多 3 次尝试机会
- 支持登出

### 2. 学生档案管理
- 添加 / 修改 / 删除学生
- 按学号精确查找
- 按姓名模糊查找
- 按班级筛选
- 显示全部学生列表
- 自动校验学号唯一性

### 3. 成绩管理
- 录入 / 修改 / 删除成绩
- 按学号或姓名查询学生全部成绩
- 分数合法性校验（0~100）
- 同一学生同一科目防止重复录入
- 显示全部成绩记录（含学生姓名、班级）

### 4. 统计与报表
- **班级排名**：按总分降序排列，使用手写快速排序
- **单科统计**：最高分、最低分、平均分、参考人数
- **按科目检索**：全体学生某科成绩排行
- **学生成绩单**：单人的各科分数、总分、平均分、班级排名

### 5. 数据管理
- 从文件加载数据（`|` 分隔符格式）
- 保存数据到文件
- 启动时自动加载，退出时可选保存

## 编译与运行

### 环境要求

- **编译器**：MinGW-w64 (GCC ≥ 8.0) 或 MSVC 2019+
- **Qt 版本**：Qt 5.12+
- **C++ 标准**：C++17

### 编译控制台版

```bash
cd src
g++ -O2 -std=c++17 main.cpp GradeSystem.cpp Student.cpp Grade.cpp FileManager.cpp -o GradeSystem.exe
./GradeSystem.exe
```

### 编译 GUI 版（Qt Creator）

1. 用 Qt Creator 打开 `src/QtGradeSystem.pro`
2. 选择 Release 构建配置
3. 点击构建 → 运行

或使用命令行：

```bash
cd src
qmake QtGradeSystem.pro
make
./QtGradeSystem.exe
```

### 直接运行

- **GUI 版**：双击 `bin/QtGradeSystem.exe`（需要安装 Qt 运行时）
- **带运行时的完整包**：打开 `打包程序/` 文件夹，双击 `QtGradeSystem.exe` 即可运行（无需安装 Qt）

## 数据文件格式

### students.txt
```
学号|姓名|班级
2024001|张三|计算机2401
2024002|李四|计算机2401
```

### grades.txt
```
学号|科目|分数
2024001|高等数学|92
2024001|大学英语|85
```

## 性能测试

对系统进行了全面的性能基准测试，具体结果见 `docs/performance_report.txt`。

### 测试关键发现

| 数据规模 | 学生数 | 成绩数 | 班级排名耗时 | 学科统计耗时 |
|----------|--------|--------|-------------|-------------|
| 小规模 | 100 | 500 | 7.5 ms | 15.3 ms |
| 中规模 | 500 | 2,500 | 97 ms | 181 ms |
| 大规模 | 1,500 | 7,500 | 847 ms | 1,560 ms |

### 排序算法对比
手写快速排序对随机数据表现良好（1000条 0.34ms），但因固定选取首元素为 pivot，对已有序数据会退化为 O(n²)：
- 20,000 条随机数据：18 ms
- 20,000 条有序数据：**550 ms**（退化 30 倍）

### 主要瓶颈

1. `calculateTotalScores` 和 `getSubjectStats` — O(n×g) 双重循环，最大耗时来源
2. `findStude O(n) 线性查找，建议改用 `unordered_map`
3. 手写快排 pivot 策略 — 有序数据退化严重

## 注意事项

1. 数据文件默认位于 `data/` 目录，程序使用相对路径 `data/students.txt` 和 `data/grades.txt`
2. 程序启动时自动加载已有数据，若文件不存在会提示错误但不影响使用
3. `打包程序/` 中的 Qt DLL 文件较大（约 50MB），仅供无 Qt 环境的用户运行参考
4. 项目使用 Qt 5 开发，与 Qt 6 不完全兼容
