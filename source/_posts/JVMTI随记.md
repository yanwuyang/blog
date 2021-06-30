---
title: JVMTI随记
date: 2021-04-10 22:48:20
comments: true 
categories: JVM
toc: false
---
JVM工具接口（JVM TI）是供工具使用的本机编程接口。它提供了一种检查状态和控制Java虚拟机中运行的应用程序执行的方法。JVMTI支持需要访问JVM状态的各种工具。

<!--more-->

### jvmti_ext_ex.cpp
```C++

#include <jvmti.h>
#include <stdio.h>
#include <memory.h>
#include <string.h>

void printStackTrace(JNIEnv* env, jobject exception) {
	jclass throwable_class = (*env).FindClass("java/lang/Throwable");
	jmethodID print_method = (*env).GetMethodID(throwable_class, "printStackTrace", "()V");
	(*env).CallVoidMethod(exception, print_method);
}

void notify(JNIEnv* env){
       jclass notify_class = env->FindClass("Demo");
       jmethodID notify_method = env->GetStaticMethodID(notify_class, "jvm_notify", "()V");
       env->CallStaticObjectMethod(notify_class,notify_method);
}

void printLocalVar(jvmtiEnv *jvmti,JNIEnv* env, jthread thread, jmethodID method,jobject obj){
      jint max_ptr;
      jvmtiError error= jvmti->GetMaxLocals(method,&max_ptr);
      if(error != JVMTI_ERROR_NONE) {
        fprintf(stderr, "ERROR: Unable to GetMaxLocals JVMTI");
        return;
      }
     // char *method_name;

      //printf("loclasMax:%d\n",max_ptr);
      jint entry_count_ptr;
      jvmtiLocalVariableEntry *table_ptr;
      error = jvmti->GetLocalVariableTable(method,&entry_count_ptr,&table_ptr);
      if(error != JVMTI_ERROR_NONE) {
        fprintf(stderr, "ERROR: Unable to GetLocalVariableTable JVMTI error %d \n",error);
        return;
      }
     // printf("entry_count_ptr:%d\n",entry_count_ptr); 
      for(int i=0;i<entry_count_ptr;i++){
         //printf("name:%s slot:%d\n",table_ptr->name,table_ptr->slot);
         jint value_ptr;
         jvmti->GetLocalInt(thread,0,i,&value_ptr);
         printf("name:%s slot:%d value %d\n",table_ptr->name,table_ptr->slot,value_ptr);
         table_ptr++;
      }

}
/*
 *thread 发生异常的线程
 *method 发生异常的方法
 *location 发生异常的位置
 *exception 发生的异常
 *catch_method 捕获异常的方法 如果没有catch 则为null
 *catch_location 捕获异常的位置 如果没有catch 则为0
 */
void JNICALL Callback_JVMTI_EVENT_EXCEPTION (jvmtiEnv *jvmti_env,
	JNIEnv* jni_env,
	jthread thread,
	jmethodID method,
	jlocation location,
	jobject exception,
	jmethodID catch_method,
	jlocation catch_location) {

	//printf("loaded class name=%s\n ", "run in Callback_JVMTI_EVENT_EXCEPTION method");
        char* class_name;
        jclass exception_class = jni_env->GetObjectClass(exception);
        jvmti_env->GetClassSignature(exception_class, &class_name, NULL);
        printf("Exception: %s %ld %ld\n", class_name,location,catch_location);	
        printStackTrace(jni_env, exception);

        if(strcmp(class_name,"Ljava/lang/ClassNotFoundException;")
            && strcmp(class_name,"Ljava/lang/NoSuchFieldError;")
            && strcmp(class_name,"Ljava/lang/NoSuchMethodException;")){
             notify(jni_env);
             printLocalVar(jvmti_env,jni_env,thread,method,exception);
        }
}

void JNICALL Callback_JVMTI_EVENT_EXCEPTIONCATCH(jvmtiEnv *jvmti_env,
            JNIEnv* jni_env,
            jthread thread,
            jmethodID method,
            jlocation location,
            jobject exception){

        char* class_name;
        jclass exception_class = jni_env->GetObjectClass(exception);
        jvmti_env->GetClassSignature(exception_class, &class_name, NULL);
       //printf("Exception: %s %ld\n", class_name,location);
       //printStackTrace(jni_env, exception);


}

void JNICALL Callback_JVMTI_EVENT_METHODENTRY(jvmtiEnv *jvmti_env,
            JNIEnv* jni_env,
            jthread thread,
            jmethodID method){
       //printf("method entry;");
}

void JNICALL Callback_JVMTI_EVENT_METHODEXIT(jvmtiEnv *jvmti_env,
            JNIEnv* jni_env,
            jthread thread,
            jmethodID method,
            jboolean was_popped_by_exception,
            jvalue return_value){
      //printf("method exit;");
}

JNIEXPORT jint JNICALL Agent_OnLoad(JavaVM *vm, char *options, void *reserved){
   fprintf(stderr,"agent onload\n"); 
   jvmtiEnv *jvmti = NULL;
   //获取JVMTI environment
   jint erno = vm->GetEnv((void **)&jvmti, JVMTI_VERSION_1_1);
   if (erno != JNI_OK) {
        fprintf(stderr, "ERROR: Couldn't get JVMTI environment");
        return JNI_ERR;
   }
   //注册功能
   jvmtiCapabilities capabilities;
   (void)memset(&capabilities, 0, sizeof(jvmtiCapabilities));
   capabilities.can_generate_exception_events=1;
   capabilities.can_access_local_variables = 1;
   capabilities.can_generate_method_entry_events=1;
   capabilities.can_generate_method_exit_events=1;
   
   jvmtiError error = jvmti->AddCapabilities(&capabilities);
   if(error != JVMTI_ERROR_NONE) {
	fprintf(stderr, "ERROR: Unable to AddCapabilities JVMTI");
	return  error;
   } 
   //设置JVM事件 (JVMTI_EVENT_EXCEPTION) 回调
   jvmtiEventCallbacks ex_callbacks;
   ex_callbacks.Exception = &Callback_JVMTI_EVENT_EXCEPTION;
   ex_callbacks.ExceptionCatch = &Callback_JVMTI_EVENT_EXCEPTIONCATCH;
   ex_callbacks.MethodEntry = &Callback_JVMTI_EVENT_METHODENTRY;
   ex_callbacks.MethodExit = &Callback_JVMTI_EVENT_METHODEXIT;
   error = jvmti->SetEventCallbacks(&ex_callbacks, (jint)sizeof(ex_callbacks));
   if(error != JVMTI_ERROR_NONE) {
	fprintf(stderr, "ERROR: Unable to SetEventCallbacks JVMTI!");		
	return error;
   }
   //设置事件通知
   error = jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_EXCEPTION, (jthread)NULL);
   error = jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_EXCEPTION_CATCH, (jthread)NULL);
   error = jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_METHOD_EXIT, (jthread)NULL);
   error = jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_METHOD_ENTRY, (jthread)NULL);
   if(error != JVMTI_ERROR_NONE) {
	fprintf(stderr, " ERROR: Unable to SetEventNotificationMode JVMTI!,the error code=%d",error);
	return  error;
   }
   
   return JNI_OK;
}

JNIEXPORT void JNICALL Agent_OnUnload(JavaVM *vm){
   fprintf(stderr,"agent onUnload\n");
}

```
### build.sh

```shell

#!/bin/sh
BASE_HOME=`pwd`
INCLUDES="-I/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/include -I/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/include/linux"
g++ $BASE_HOME/jvmti_ext_ex.cpp $INCLUDES -Wall -Wno-deprecated -fPIC --share -o $BASE_HOME/jvmti_ext_ex.so

```

### Demo.java
```java

public class Demo{

   public static void main(String[] args){
     
      System.out.println("hello world");
      int i=0;
      try{
         i=100/i;
      }catch(Exception e){
    
      }
      new Thread(()->{
         msg(1);
      }).start();
    
      // msg(10);
   }
   
   public static int msg(int i){
      return i+1;
   }
  
   public static void error(int i){
      i++;
      throw new RuntimeException();
   }
  
   public static void jvm_notify(){
       System.out.println("notify");
   }
 
}

```

### run.sh


```shell
#!/bin/sh

java -agentpath:jvmti_ext_ex.so  Demo
#java -agentlib:hprof=cpu=samples,interval=1,depth=3 Demo

```