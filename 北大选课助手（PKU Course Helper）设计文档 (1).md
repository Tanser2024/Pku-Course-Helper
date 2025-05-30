[TOC]

# 北大选课助手（PKU Course Helper）设计文档

## 0.关于我们

我们是**Lazy Coders**

## 1 .项目背景

每每来到选课季，p大学生就会面临：

**课程信息分散** 选课系统提供信息笼统，需要从非官方测评网站，树洞等整理信息

**评价主观性强** 不同水平选手提供的课程从测评大相径庭，难以参考

**课程规划困难** 通识课，公选课众多，难以找到自己感兴趣的课或是不知道如何抉择

于是，PKU Course Helper 诞生了！我们希望通过收集整理非官方课程测评网站和选课网提供的课程相关信息，结合课程推荐算法，个性化帮助p大学生选课，为各位提供你或许感兴趣的课程

## 2 .功能需求

### **核心功能**

✅ **课程查询**

- 查询官方课程信息（名称、时间、教师、学分）
- 显示课程评分（ workload、给分、趣味性 ）

✅ **智能推荐**

- 输入专业/兴趣标签（如“想划水”“想学硬核知识”），生成推荐课表
- **“防踩雷”模式**：自动过滤低分课程

✅ **冲突检测**

- 自动检测时间冲突，避免“鬼畜课表”

### **Bonus功能（如果时间够）**

✨ **课程评价社区**（学生匿名点评，可以用来丰富数据库）
✨ **本地课表导出**（PDF/QT的打印功能）

## **3. 技术方案**

### **数据从哪里来？**

- **爬虫**：从北大教务课程查询系统（需模拟登录）抓课程列表 ,从课程测评网站获取课程评价信息
- **Mock数据**（如果教务系统爬不动，先造点假数据[doge]）
- **本地存储** SQLite数据库（QT自带支持）

### **前端（QT实现）**

- **UI框架**：QT Widgets（传统但稳定） + **QSS**美化
- **核心控件**：
  - `QTabWidget`：分“课程查询”“智能推荐”“我的课表”等标签页
  - `QTableView` + `QSqlTableModel`：绑定数据库，显示课程列表
  - `QCalendarWidget`：可视化展示课表时间冲突

### **后端逻辑**

采用**多标签渐进筛选**的方法，并预先将课程分类，按照分类后的课程数量来对标签进行排序

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>

using namespace std;

// 课程类
class Course {
public:
    int id;
    string name;
    vector<string> tags; // 课程标签
    vector<string> information;//课程信息
    
    Course(int _id, string _name, vector<string> _tags,vector<string> _information) 
        : id(_id), name(_name), tags(_tags),information(_information) {}
};

// 课程推荐系统类
class CourseRecommender {
private:
    vector<Course> allCourses;
    unordered_map<string, vector<int>> tagToCourseIds; // 每个标签到所有拥有该标签课程ID的索引

public:
    // 添加课程到系统
    void addCourse(const Course& course) {
        allCourses.push_back(course);
        // 更新倒排索引
        for (const string& tag : course.tags) {
            tagToCourseIds[tag].push_back(course.id);
        }
    }

    // 根据多个标签推荐课程
    vector<Course> recommendCourses(const vector<string>& userTags) {
        if (userTags.empty() || allCourses.empty()) {
            return {};
        }

        // 第一步：获取第一个标签对应的课程ID集合
        vector<int> resultIds = tagToCourseIds[userTags[0]];

        // 第二步：逐步与其他标签取交集
        for (size_t i = 1; i < userTags.size(); ++i) {
            const vector<int>& currentIds = tagToCourseIds[userTags[i]];
            vector<int> temp;
            
            // 求交集
            set_intersection(
                resultIds.begin(), resultIds.end(),
                currentIds.begin(), currentIds.end(),
                back_inserter(temp)
            );
            
            // 如果交集为空，提前终止
            if (temp.empty()) {
                return {};
            }
            
            resultIds = move(temp);
        }

        // 第三步：根据ID查找完整课程信息
        vector<Course> recommendedCourses;
        for (int id : resultIds) {
            auto it = find_if(allCourses.begin(), allCourses.end(),
                [id](const Course& c) { return c.id == id; });
            if (it != allCourses.end()) {
                recommendedCourses.push_back(*it);
            }
        }

        return recommendedCourses;
    }
};


```

## **4. 进度安排**

| 阶段 | 任务                               | 时间     |
| :--- | :--------------------------------- | :------- |
| 1    | QT环境搭建 + 基础UI设计            | Week 1   |
| 2    | 爬虫数据导入 + 冲突检测逻辑        | Week 2   |
| 3    | 智能推荐                           | Week 3,4 |
| 4    | if (LeftTime !=0）{界面美化；测试} | Week 5   |

------

## **5. 可能的坑 & 应对方案**

 **坑1：教务系统反爬**
→ 方案：采用模拟登录，或者直接找教务处要数据（理直气壮：“为了同学们的利益！”）

 **坑2：缺少用户评价数据**
→ 方案：自己编点搞笑评价并随机放出

 **坑3：推荐算法太智障**
→ 方案：~~直接规则匹配（if-else），人工实现智能~~  查阅优秀算法，学习并实现

## **6. 项目愿景**

**短期**：帮北大学生选课不踩雷，争取成为“选课季救命神器”。
~~**长期**：扩张到其他高校，改名叫“全国大学生选课助手”，然后被教务处封杀。~~







