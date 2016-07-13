---
layout: post
title:  "Dynamic Input Field Pattern"
date:   2016-07-13 21:57:00 +0700
categories: DevOps
tags:
 - Apex
 - SalesForce.com
---
Requirement มีอยู่ว่า อยากได้ Web Page ที่มี Feature หนึ่งที่สามารถ Add Item ได้เรื่อยๆ แบบไม่มีกำหนดจำนวนที่ชัดเจน คำถามคือ ถ้าเรารู้จำนวนที่ชัดเจนแล้วเราจะทำ `<input>` ยังไง คำตอบคือเราต้องให้ `Ajax` ช่วย แล้วทำการ repeat `<input>` tag จากนั้นก็เก็บมันในรูปแบบ List ซะ

เรามาดูในส่วนของ Controller กันก่อน
{% highlight java %}
public with sharing class Store {
    public class Itemx {
        public Decimal itemnumber {get; set;}
        public String description {get; set;}
        public Id tag {get; set;}
    }

    public List<Itemx> sowItems {get; set;}

    public Store() {
        sowItems = new List<Itemx>();

        // First item
        Itemx item = new Itemx();
        sowItems.add(item);
    }

    public void addNewItem() {      
        Itemx item = new Itemx();
        sowItems.add(item);
    }

    public void updateTagid() {
        for(Itemx i : sowItems) {
            i.tag = tagId;
        }

        insert sowItems;
    }
}
{% endhighlight %}
ทำการสร้าง Sub-Class ขึ้นมาก่อน จากนั้นก็เก็บลง List<Object> แบบ Objects ต่อมาเตรียม method เพื่อ Add New Item ลง List ถ้ามีการเรียกจากหน้า Web List ก็จะถูกเพิ่มขึ้นที่ละ 1 นั้นเอง

นี้เป็นหน้า Web แบบง่ายๆ เขียนด้วย Apex
{% highlight html %}
<apex:page controller="Store">
    <apex:composition template="Global_Application_Template_VF">
        <apex:define name="header">
            <title>Simple Page</title>            
        </apex:define>
        <apex:define name="body">
            <apex:pageBlock title="Area 51">
                <apex:form>
                    <apex:outputPanel id="form">
                    <table  border="1" cellspacing="0" cellpadding="0">
                        <thead>
                            <tr>
                                <th>#Item</th>
                                <th>Description</th>
                            </tr>
                        </thead>
                        <tbody>
                                <apex:variable var="cnt" value="{!1}" />
                                <apex:repeat var="item" value="{!sowItems}">
                                    <tr>
                                        <td><apex:OutputText value="{!cnt}"/></td>
                                        <td><apex:inputText value="{!item.description}" /></td>
                                    </tr>
                                    <apex:variable var="cnt" value="{!cnt+1}"/>
                                </apex:repeat>
                        </tbody>
                    </table>
                    </apex:outputPanel>
                    <apex:commandButton action="{!addNewItem}" value="Add New" rerender="form" />
                    <apex:commandButton action="{!updateTagid}" value="Update All" rerender="form" />
                </apex:form>
            </apex:pageBlock>
        </apex:define>
    </apex:composition>
</apex:page>
{% endhighlight %}
การทำ rerender ก็เห็นกับทำ Ajax ที่ส่วนนี้นั้นเอง

![Input 1](/images/dm-input-1.png)


![Input 2](/images/dm-input-2.png)
