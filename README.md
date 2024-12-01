# NotePad
如下是我更新以后的NotePad

/-------时间戳实现部分

1.首先添加COLUMN_NAME_MODIFICATION_DATE字段，用于存储时间戳  


![image](https://github.com/user-attachments/assets/468efb1c-2304-418b-964b-a7a2019e2390)

2.然后在视图中添加显示时间戳的代码  

![image](https://github.com/user-attachments/assets/b3ed13d9-c713-403e-af73-00d5fced9145)

3.将时间戳格式改为年月日

![image](https://github.com/user-attachments/assets/b90b9096-3c4c-4fb5-a3cc-8e250d658d26)

4.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/7e5343c6-4b57-4c9c-9f5b-4546dad8dd6f)


/--------搜索功能实现部分

1.在onCreateOptionsMenu中添加搜索功能  

![image](https://github.com/user-attachments/assets/d5295a69-d722-45da-b48d-052361001817)


2.包括searchNotes方法  

![image](https://github.com/user-attachments/assets/ea8e03b4-b69d-4e61-b224-a7147857fd04)

3.更新Menu资源  

![image](https://github.com/user-attachments/assets/eb565c5b-715c-48ea-8520-37c8471726ff)

4.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/50734b7a-dc4f-4a15-ae04-3f697fe8c719)

/--------UI美化实现部分

1.覆写getview，该功能能够实现随机生成不同背景颜色的listview，同时我让RGB在一个较为柔和的区间，以免视觉冲击过大  

![image](https://github.com/user-attachments/assets/4a408810-da6b-4c53-90ae-9f7b3556a96e)

2.包括getContrastingColor取对比颜色方法，能够让标题和时间戳随背景随机变化而更加明显  

![image](https://github.com/user-attachments/assets/a03e3f35-19ec-42d3-be02-42ab6ee02d1e)

3.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/6e9929b6-f3fe-4fac-809b-93d039c20963)

/--------导出笔记功能实现部分

1.添加菜单项  

![image](https://github.com/user-attachments/assets/f5503937-87a0-4d03-ab05-aad690d72539)

2.在onOptionsItemSelected方法中处理导出菜单项  

![image](https://github.com/user-attachments/assets/df80b57b-7e70-4f22-ad3e-4e90f309a16f)

3.实现exportNotes方法  

![image](https://github.com/user-attachments/assets/78f6789a-1c33-4f5a-b283-df5f1bf00398)

4.修改PROJECTION包括笔记内容  

![image](https://github.com/user-attachments/assets/9ec14604-d37b-4aca-afe7-dcf93bb28177)

5.确保权限，在AndroidManifest.xml中添加文件写入权限  

![image](https://github.com/user-attachments/assets/4d8cb696-2670-4ddd-b042-1642081913f3)

6.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/a461f038-53c3-4852-9b2d-35345e88d6e9)






