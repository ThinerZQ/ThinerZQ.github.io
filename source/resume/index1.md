title: ThinerZQ's Resume
date: 2016-01-28 14:02:04
update: 2016-02-03 18:08:32
layout: post
---

```java
package com.zq.resume;

import java.time.LocalDate;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;

/**
 * Created with IntelliJ IDEA
 * Date: 2016/2/3
 * Time: 20:04
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://www.thinerzq.me</a>
 * Email: 601097836@qq.com
 */

public class Resume {

    public static void main(String[] args) {

        Resume resume = new Resume();

        //个人基本信息
        resume.setName(" 郑  强 ")
                .setAge(23)
                .setGender("男")
                .setLocation("广东，广州")
                .setPoliticalStatus("中共党员")
                .setHighestAcademic("硕士")
                .setUniversity("中山大学")
                .setPhoneNumber("15626234928")
                .setEmail("601097836@qq.com")
                .setWorkExperience("应届毕业生工作经历")
                .setHomePage("http://www.thinerzq.me")
                .setGitHub("https://www.github.com/ThinerZQ")
                .setJobIntention("java开发")
                .setHobby(new String[]{"篮球", "足球", "羽毛球", "网球", "阅读", "摄影", "交友", "轮滑", "骑行"})
                .setCertificate(new String[]{"英语六级", "软件设计师"})
                .setSelfEvaluation("我是一个学习能力强、" +
                        "对新事物充满好奇、" +
                        "能独立解决问题、" +
                        "有责任心、" +
                        "善于与人沟通、" +
                        "热爱运动、" +
                        "热爱生活的文艺青年")
                .setSelfDescription("研究生期间的研究方向是状态机工作流，" +
                        "目前根据开源项目Apache Commons SCXML编写了一个状态机工作流引擎BOWorkflow，" +
                        "并一直在维护，" +
                        "同时自学过一段时间Andriod开发，" +
                        "对数据挖掘特别感兴趣。");


        //个人教育经历
        EducationExperience university = new EducationExperience();
        university.setBeginDate(LocalDate.of(2010, 9, 30))
                .setEndDate(LocalDate.of(2014, 6, 17))
                .setUniversity("厦门理工学院")
                .setMajor("软件工程")
                .setDegree("本科");
        EducationExperience postgraduate = new EducationExperience();
        postgraduate.setBeginDate(LocalDate.of(2014, 8, 23))
                .setEndDate(LocalDate.now())
                .setUniversity("中山大学")
                .setMajor("软件工程")
                .setDegree("硕士");

        //个人校园经历
        CampusActivity ccnaTrain = new CampusActivity();
        ccnaTrain.setActivityBeginDate(LocalDate.of(2012, 7, 10))
                .setActivityEndDate(LocalDate.of(2012, 8, 23))
                .setActivityName("硕翔CCNA培训")
                .setActivityDescription("这次培训经历是硕翔公司与学校合作，" +
                        "免费提供，" +
                        "最后我们利用公司的大型路由器和交换机完成了一个小型局域网的搭建.");

        CampusActivity studentUnion = new CampusActivity();
        studentUnion.setActivityBeginDate(LocalDate.of(2012, 9, 5))
                .setActivityEndDate(LocalDate.of(2013, 9, 28))
                .setActivityName("院学生会学习部部长")
                .setActivityDescription("担任院学生会学习部部长，" +
                        "这段经历教会了我很东西，" +
                        "一方面锻炼了自己为人处事的能力、" +
                        "提升了与人沟通的技巧，" +
                        "另一方面也培养了我的责任心。");

        CampusActivity familyEducation = new CampusActivity();
        familyEducation.setActivityBeginDate(LocalDate.of(2015,5,14))
                .setActivityEndDate(LocalDate.now())
                .setActivityName("C语言家教")
                .setActivityDescription("负责某高中生的C语言教学。");

        //个人获奖情况
        Award softwareDesignAward = new Award();
        softwareDesignAward.setAwardDate(LocalDate.of(2013, 7, 28))
                .setAwardName("院系软件设计大赛一等奖");

        Award nationalEncouragementScholarship = new Award();
        nationalEncouragementScholarship.setAwardDate(LocalDate.of(2013, 10, 11))
                .setAwardName("国家励志奖学金");

        Award schoolAward = new Award();
        schoolAward.setAwardDate(LocalDate.of(2015, 10, 16))
                .setAwardName("中山大学三等奖学金");

        //个人项目经历
        ProjectExperience personalManagementSystem = new ProjectExperience();
        personalManagementSystem.setProjectBeginDate(LocalDate.of(2013, 3, 1))
                .setProjectEndDate(LocalDate.of(2013, 5, 28))
                .setProjectName("人事管理系统")
                .setProjectPosition("负责人")
                .setProjectDescription("这是《软件工程》课上老师提出的一个项目，" +
                        "由我担任小组长。" +
                        "针对老师提出的一系列稀奇古怪的需要（页面上需要良好的的Ajax交互和丰富的Jquery动画操作），" +
                        "建立一套web人事管理系统，" +
                        "并最终获得了班级第一名");

        ProjectExperience onlineShop = new ProjectExperience();
        onlineShop.setProjectBeginDate(LocalDate.of(2013, 7, 2))
                .setProjectEndDate(LocalDate.of(2013, 7, 30))
                .setProjectName("刀刀狗在线商城")
                .setProjectPosition("技术指导")
                .setProjectDescription("这是我们2013是暑假实训的是一个项目，" +
                        "主要任务就是编写一个在线商城，" +
                        "我们模仿天猫主页，" +
                        "实现了一个小型的在线商城，" +
                        "并最终获得了院系一等奖。");

        ProjectExperience retiredStaffManagementSystem = new ProjectExperience();
        retiredStaffManagementSystem.setProjectBeginDate(LocalDate.of(2014,4,1))
                .setProjectEndDate(LocalDate.of(2013, 6, 22))
                .setProjectName("厦门市退休人员管理系统")
                .setProjectPosition("负责人")
                .setProjectDescription("这是我本科的毕业设计，" +
                        "整个项目基于SSH2架构。" +
                        "主要功能是离退休人员的信息管理。")
                .setProjectDifficulties(new ArrayList<String>() {{
                    add("档案的在线扫描与归档");
                    add("档案的在线预览");
                }});

        Skill windows = new Skill();



        List<EducationExperience> educationExperiences = new LinkedList<>();
        educationExperiences.add(postgraduate);
        educationExperiences.add(university);

        List<CampusActivity> campusActivities = new LinkedList<>();
        campusActivities.add(ccnaTrain);
        campusActivities.add(studentUnion);
        campusActivities.add(familyEducation);

        List<Award> awards = new LinkedList<>();
        awards.add(softwareDesignAward);
        awards.add(nationalEncouragementScholarship);
        awards.add(schoolAward);

        List<ProjectExperience> projectExperiences  = new LinkedList<>();
        projectExperiences.add(personalManagementSystem);
        projectExperiences.add(onlineShop);
        projectExperiences.add(retiredStaffManagementSystem);


        resume.setEducationExperiences(educationExperiences)
                .setCampusActivities(campusActivities)
                .setAwardses(awards)
                .setProjectExperiences(projectExperiences);





        System.out.println(resume);
    }

    private String name;
    private int age;
    private String gender;
    private String location;
    private String politicalStatus;
    private String highestAcademic;
    private String university;
    private String phoneNumber;
    private String email;
    private String workExperience;
    private String homePage;
    private String gitHub ;
    private String jobIntention;
    private String[] hobby;
    private String[] certificate;
    private String selfEvaluation;
    private String selfDescription;
    private List<EducationExperience> educationExperiences;
    private List<CampusActivity> campusActivities;
    private List<Award> awardses;
    private List<ProjectExperience> projectExperiences;
    private List<Skill> skills;


    @Override
    public String toString() {
        return "Resume{\n" +
                "\t name='" + name + "\'\n" +
                "\t age=" + age + "\n"+
                "\t gender='" + gender + "\'\n" +
                "\t location='" + location + "\'\n" +
                "\t politicalStatus='" + politicalStatus + "\'\n" +
                "\t highestAcademic='" + highestAcademic + "\'\n" +
                "\t university='" + university + "\'\n" +
                "\t phoneNumber='" + phoneNumber + "\'\n" +
                "\t email='" + email +"\'\n" +
                "\t workExperience='" + workExperience +"\'\n" +
                "\t homePage='" + homePage +"\'\n" +
                "\t gitHub='" + gitHub + "\'\n" +
                "\t jobIntention='" + jobIntention + "\'\n" +
                "\t hobby=" + Arrays.toString(hobby) +"\n"+
                "\t certificate=" + Arrays.toString(certificate) +"\n"+
                "\t selfEvaluation='" + selfEvaluation + "\'\n" +
                "\t selfDescription='" + selfDescription + "\'\n" +
                "\t educationExperiences=" + educationExperiences +"\n"+
                "\t campusActivities=" + campusActivities +"\n"+
                "\t awardses=" + awardses +"\n"+
                "\t projectExperiences=" + projectExperiences +"\n"+
                "\t skills=" + skills +"\n"+
                '}';
    }

    public String getName() {
        return name;
    }

    public Resume setName(String name) {
        this.name = name;
        return this;
    }

    public int getAge() {
        return age;
    }

    public Resume setAge(int age) {
        this.age = age;
        return this;
    }

    public String getGender() {
        return gender;
    }

    public Resume setGender(String gender) {
        this.gender = gender;
        return this;
    }

    public String getLocation() {
        return location;
    }

    public Resume setLocation(String location) {
        this.location = location; return this;
    }

    public String getPoliticalStatus() {
        return politicalStatus;
    }

    public Resume setPoliticalStatus(String politicalStatus) {
        this.politicalStatus = politicalStatus; return this;
    }

    public String getHighestAcademic() {
        return highestAcademic;
    }

    public Resume setHighestAcademic(String highestAcademic) {
        this.highestAcademic = highestAcademic; return this;
    }

    public String getUniversity() {
        return university;
    }

    public Resume setUniversity(String university) {
        this.university = university; return this;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }

    public Resume setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber; return this;
    }

    public String getEmail() {
        return email;
    }

    public Resume setEmail(String email) {
        this.email = email; return this;
    }

    public String getWorkExperience() {
        return workExperience;
    }

    public Resume setWorkExperience(String workExperience) {
        this.workExperience = workExperience; return this;
    }

    public String getHomePage() {
        return homePage;
    }

    public Resume setHomePage(String homePage) {
        this.homePage = homePage; return this;
    }

    public String getGitHub() {
        return gitHub;
    }

    public Resume setGitHub(String gitHub) {
        this.gitHub = gitHub; return this;
    }

    public String getJobIntention() {
        return jobIntention;
    }

    public Resume setJobIntention(String jobIntention) {
        this.jobIntention = jobIntention; return this;
    }

    public String getSelfDescription() {
        return selfDescription;
    }

    public Resume setSelfDescription(String selfDescription) {
        this.selfDescription = selfDescription; return this;
    }

    public String[] getHobby() {
        return hobby;
    }

    public Resume setHobby(String[] hobby) {
        this.hobby = hobby; return this;
    }

    public String[] getCertificate() {
        return certificate;
    }

    public Resume setCertificate(String[] certificate) {
        this.certificate = certificate; return this;
    }

    public String getSelfEvaluation() {
        return selfEvaluation;
    }

    public Resume setSelfEvaluation(String selfEvaluation) {
        this.selfEvaluation = selfEvaluation; return this;
    }

    public List<EducationExperience> getEducationExperiences() {
        return educationExperiences;
    }

    public Resume setEducationExperiences(List<EducationExperience> educationExperiences) {
        this.educationExperiences = educationExperiences;
        return this;
    }

    public List<CampusActivity> getCampusActivities() {
        return campusActivities;
    }

    public Resume setCampusActivities(List<CampusActivity> campusActivities) {
        this.campusActivities = campusActivities; return this;
    }

    public List<Award> getAwardses() {
        return awardses;
    }

    public Resume setAwardses(List<Award> awardses) {
        this.awardses = awardses; return this;
    }

    public List<ProjectExperience> getProjectExperiences() {
        return projectExperiences;
    }

    public Resume setProjectExperiences(List<ProjectExperience> projectExperiences) {
        this.projectExperiences = projectExperiences; return this;
    }

    public List<Skill> getSkills() {
        return skills;
    }

    public Resume setSkills(List<Skill> skills) {
        this.skills = skills; return this;
    }
}

```