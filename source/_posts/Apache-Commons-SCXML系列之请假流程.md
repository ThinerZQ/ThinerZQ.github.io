title: Apache Commons-SCXML系列之请假流程
date: 2016-02-01 21:34:34
tags:
 scxml例子
categories:
 Commons-SCXML
---

首先分析自己的业务逻辑，画好状态图

# 请假流程状态图

![请假流程状态图，画图工具EA](http://img.blog.csdn.net/20151202150236801)

<!--more-->

# 根据状态图编写xml文件

```xml
<?xml version="1.0"?>
	<scxml xmlns="http://www.w3.org/2005/07/scxml"
       version="1.0"
       datamodel="jexl"
       initial="filling">

    <!-- 请假单需要的数据-->
    <datamodel>
        <data id="applicant" expr=""></data>
        <data id="reason" expr=""></data>
        <data id="from" expr=""></data>
        <data id="to" expr=""></data>
        <data id="departmentApprove" expr="false"></data>
        <data id="personnelApprove" expr="false"></data>
    </datamodel>

    <state id="filling">
        <onentry>

        </onentry>
        <!--当填写完了表单，将外界的数据通过,系统变量_event.data传进来，data其实是一个map-->
        <transition event="fill.end" target="approving">
            <assign location="applicant" expr="_event.data.name"></assign>
            <assign location="reason" expr="_event.data.reason"></assign>
            <assign location="from" expr="_event.data.from"></assign>
            <assign location="to" expr="_event.data.to"></assign>
            <!--调用leaveEntity的填表函数，在这个函数里面，我们可以操作做数据持久化-->
            <!--个人觉得这种方式和领域驱动设计很像，欢迎和我交流-->
            <script>
                leaveEntity.fillForm(applicant,reason,from,to)
            </script>

        </transition>
    </state>

    <!--这是一个复合状态：一个大状太-->
    <state id="approving">
        <initial>
            <transition target="departmentApproving">

            </transition>
        </initial>
        <!--部门经理审批状态-->
        <state id="departmentApproving">
            <onentry></onentry>
            <!--如果部门经理同意，就将departmentApprove的值赋值为true，然后我们可以执行持久化的数据操作（我这里只是简单的输出）-->
            <transition event="approve" target="personnelApproving">
                <assign location="departmentApprove" expr="true"></assign>
                <script>
                    leaveEntity.departmentApprove(departmentApprove)
                </script>
            </transition>
        </state>
        <!--人事经理审批状态-->
        <state id="personnelApproving">
            <!--如果人事经理同意，就将personnelApprove的值赋值为true，然后我们可以执行持久化的数据操作（我这里只是简单的输出）-->
            <transition event="approve" target="approveEnd">
                <assign location="personnelApprove" expr="true"></assign>
                <script>
                    leaveEntity.personnelApprove(personnelApprove)
                </script>
            </transition>
        </state>

        <final id="approveEnd"></final>

        <!--这个转移的事件“event.state.approving”是当当前复合状态到达 <final> 节点的时候框架自动生成的。-->
        <transition event="done.state.approving" target="approved" />
        <!--这个转移事件，是只要任何一个经理拒绝了请求，就转向被拒绝状态-->
        <transition event="reject" target="rejected">

        </transition>
    </state>
    <!--已同意状态，进入状态的时候，可以发送邮件给相应的用户-->
    <state id="approved">
        <onentry>
            <script>
                leaveEntity.sendEmail()
            </script>
        </onentry>
        <!--一个eventless（自动转移），执行完了<onentry>里面的事件就转移了-->
        <transition target="archiving" ></transition>
    </state>
    <!--被拒绝状态，进入状态的时候，可以发送邮件给相应的用户-->
    <state id="rejected">
        <onentry>
            <script>
                leaveEntity.sendEmail()
            </script>
        </onentry>
        <!--申请者选择继续修改申请信息的时候的转移，-->
        <transition event="goFilling" target="filling"></transition>
        <!--申请者可以取消本次请假-->
        <transition event="goEnd" target="archiving"></transition>
    </state>
    <!--归档状态，进入的时候直接归档-->
    <state id="archiving">
        <onentry>
            <script>
                leaveEntity.archive()
            </script>
        </onentry>
        <transition target="end"/>
    </state>

    <!--结束状态-->
    <final id="end">
    </final>
</scxml>

```



# 编写程序、控制状态变化
<code>LeaveEntity.java</code>

```java

package leave;

public class LeaveEntity {

    
    private String appliant;
    private String reason;
    private String from ;
    private String to;
    private boolean departmentApprove;
    private boolean personnelApprove;


   
    public void fillForm(String name,String reason,String from,String to){

        this.appliant = name;
        this.reason = reason;
        this.from=from;
        this.to= to;
        System.out.println("请假人："+appliant);
        System.out.println("请假原因："+reason);
        System.out.println("开始时间："+from);
        System.out.println("结束时间："+to);
    }

  
    public void departmentApprove(boolean b){
        this.departmentApprove = b;
        System.out.println("部门经理同意");
    }
  
    public void personnelApprove(boolean b){
        this.personnelApprove =b;
        System.out.println("人事经理同意");
    }

    public void sendEmail(){
        if (departmentApprove && personnelApprove){
            System.out.println(appliant+":你好，你的请假已经通过了");
        }else{
            System.out.println(appliant+":你好，你的请假没有通过，请重新填写");
        }
    }

    public void archive(){
        System.out.println("开始归档");
    }
}

```
<code>LeaveFrame.java</code>  
``` java
package leave;

import org.apache.commons.scxml2.Context;
import org.apache.commons.scxml2.Evaluator;
import org.apache.commons.scxml2.SCXMLExecutor;
import org.apache.commons.scxml2.TriggerEvent;
import org.apache.commons.scxml2.env.SimpleErrorReporter;
import org.apache.commons.scxml2.env.jexl.JexlEvaluator;
import org.apache.commons.scxml2.io.SCXMLReader;
import org.apache.commons.scxml2.model.ModelException;
import org.apache.commons.scxml2.model.SCXML;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.net.URL;
import java.util.HashMap;
import java.util.Map;
public class LeaveFrame extends JFrame implements ActionListener {

private static final long serialVersionUID = 1L;


private JLabel applicant;
private JLabel reason;
private JLabel from;
private JLabel to;

private JTextField nameTest;
private JTextField reasonTest;
private JTextField fromTest;
private JTextField toTest;

private JButton submit;
private JButton departmentApprove;
private JButton personeelApprove;
private JButton reject;
private JButton continueFill;
private JButton archive;
private JButton start;

private SCXMLExecutor executor=null;

public LeaveFrame() {
    super("SCXML Leave");
    initUI();
}

public static void main(String[] args) {
    new LeaveFrame();
}


private void initWorkflow()  {
    
    final URL leaveApprovel = this.getClass().getResource("leaveApprove1.xml");
    
    Evaluator evaluator = new JexlEvaluator();

    
    executor = new SCXMLExecutor(evaluator, null, new SimpleErrorReporter());

    try {
        
        SCXML scxml = SCXMLReader.read(leaveApprovel);

        
        executor.setStateMachine(scxml);

        
        Context rootContext = evaluator.newContext(null);
        LeaveEntity leaveEntity = new LeaveEntity();
        rootContext.set("leaveEntity", leaveEntity);
        executor.setRootContext(rootContext);

        
        executor.go();

    }catch (Exception e){
        e.printStackTrace();
    }

    System.out.println(executor.getGlobalContext().getSystemContext().get("_sessionid"));
}


public void actionPerformed(ActionEvent event) {
    String command = event.getActionCommand();
    try {
        if ("submit".equals(command)) {

            setEnabledAndDisabled(new JComponent[]{reject,departmentApprove}, new JComponent[]{submit});

           
            Map<String, String> payloadData = new HashMap<String, String>();
            payloadData.put("name",nameTest.getText());
            payloadData.put("reason",reasonTest.getText());
            payloadData.put("from", fromTest.getText());
            payloadData.put("to", toTest.getText());
           
            executor.triggerEvent(new TriggerEvent("fill.end", TriggerEvent.SIGNAL_EVENT,payloadData));

        }else if ("departmentApprove".equals(command)){


            setEnabledAndDisabled(new JComponent[]{personeelApprove}, new JComponent[]{departmentApprove});

            executor.triggerEvent(new TriggerEvent("approve", TriggerEvent.SIGNAL_EVENT));

        }else if ("personeelApprove".equals(command)){

            setEnabledAndDisabled(new JComponent[]{}, new JComponent[]{personeelApprove,reject,departmentApprove});
            executor.triggerEvent(new TriggerEvent("approve", TriggerEvent.SIGNAL_EVENT));


        }else if ("reject".equals(command)){

            setEnabledAndDisabled(new JComponent[]{continueFill,archive}, new JComponent[]{submit,personeelApprove,reject,departmentApprove});

            executor.triggerEvent(new TriggerEvent("reject", TriggerEvent.SIGNAL_EVENT));

        }else if ("continueFill".equals(command)){

            setEnabledAndDisabled(new JComponent[]{submit}, new JComponent[]{departmentApprove, personeelApprove, reject, continueFill, archive});

            executor.triggerEvent(new TriggerEvent("goFilling", TriggerEvent.SIGNAL_EVENT));

        }else if ("archive".equals(command)){


            setEnabledAndDisabled(new JComponent[]{}, new JComponent[]{departmentApprove, personeelApprove, reject, continueFill, archive, submit});
            executor.triggerEvent(new TriggerEvent("goEnd", TriggerEvent.SIGNAL_EVENT));
        }else if ("start".equals(command)){

            setEnabledAndDisabled(new JComponent[]{submit}, new JComponent[]{departmentApprove, personeelApprove, reject, continueFill, archive, start});
            setEnabledAndDisabled(new JComponent[]{nameTest,reasonTest,fromTest,toTest}, new JComponent[]{});

            initWorkflow();

        }
    } catch (ModelException e) {
        e.printStackTrace();
    }
}


private void initUI() {

    JPanel mainPanel = new JPanel();
    mainPanel.setLayout(new BorderLayout());

    JPanel contentPanel = new JPanel();

    contentPanel.setLayout(new GridLayout(8,2));

    applicant = new JLabel("申请人：");
    nameTest = new JTextField(10);

    reason = new JLabel("原因：");
    reasonTest = new JTextField(50);


    from = new JLabel("开始时间：");
    fromTest = new JTextField(10);


    to = new JLabel("结束时间：");
    toTest = new JTextField(10);

    start = createButton("start","请假");
    submit = createButton("submit","Submit");
    departmentApprove= createButton("departmentApprove","部门同意");
    personeelApprove= createButton("personeelApprove","人事同意");
    reject = createButton("reject","拒绝");
    continueFill = createButton("continueFill","继续填写");
    archive= createButton("archive","结束");



    setEnabledAndDisabled(new JComponent[]{start}, new JComponent[]{departmentApprove, personeelApprove, reject, continueFill, archive, submit});
    setEnabledAndDisabled(new JComponent[]{}, new JComponent[]{nameTest, reasonTest, fromTest, toTest});


    gridLayoutAdd(contentPanel,new JComponent[]{applicant,nameTest,reason,reasonTest,from,fromTest,to,toTest,start,submit,departmentApprove,personeelApprove,reject,continueFill,archive});

    mainPanel.add(contentPanel, BorderLayout.CENTER);

    setContentPane(mainPanel);

    setLocation(200, 200);
    setSize(400, 400);

    setResizable(true);
    setVisible(true);

    setDefaultCloseOperation(EXIT_ON_CLOSE);


}

private void gridLayoutAdd(JPanel content, JComponent[] components){

    for (int i = 0; i < components.length; i++) {
        content.add(components[i]);
    }

}

private JButton createButton(final String command, final String text) {
    JButton button = new JButton(text);
    button.setActionCommand(command);
    button.addActionListener(this);
    return button;
}
private void setEnabledAndDisabled(JComponent[] enabled,JComponent[] disabled){

    for (int i=0;i<enabled.length;i++){
        enabled[i].setEnabled(true);
    }
    for (int i=0;i<disabled.length;i++){
        disabled[i].setEnabled(false);
    }
}
}
```
    

# 程序分析
## 首先打开程序
<br>
![这里写图片描述](http://img.blog.csdn.net/20151203093928774)
<br>
## 然后点击请假，发起一个请假流程，图中的一串字符是当前会话的id。
<br>
![这里写图片描述](http://img.blog.csdn.net/20151203094017608)
<br>
## 一旦提交表单，就轮到部门经理审批，部门经理可以选择同意或者拒绝。
<br>
![这里写图片描述](http://img.blog.csdn.net/20151203094154555)
<br>
## 这里先演示部门同意，接下来就该到人事经理了。
<br>
![这里写图片描述](http://img.blog.csdn.net/20151203094308804)
<br>
## 这里演示人事经理拒绝，那么按照我们之前话的状态图，这时候应该到“被拒绝“状态，发起人可以更改信息，重新提交或者放弃请假。
<br>
![这里写图片描述](http://img.blog.csdn.net/20151203094427507)
## 这里假设我们继续填写，点击提交，又回到了部门经理审批，然后人事经理审批环节，假设两者都同意，然后整个流程就结束了。
<br>
![这里写图片描述](http://img.blog.csdn.net/20151203094613452)
# 总结
用状态图只要控制好event 和condition 表达复杂的行为很方便。SCXML框架只有控制状态变化的能力，如果我们能够添加上任务分派功能，那就是一个很强大的状态机工作流了。



