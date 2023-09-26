---
layout:     post
title:      Mac M2芯片编译hudi源码问题之protoc
subtitle:   hudi源码编译
date:       2023-9-14
author:     zhmingyong
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Mac
    - M2
    - hudi源码编译
---

# Mac M2芯片编译hudi源码问题之protoc

解决部署Hudi时遇到的 Error extracting protoc for version 3.21.5: Unsupported platform: protoc-xxx.exe 问题

在部署Hudi时遇到以下不支持protoc的问题：

```bash
[ERROR] Failed to execute goal com.github.os72:protoc-jar-maven-plugin:3.11.4:run (default) on project hudi-kafka-connect: Error extracting protoc for version 3.21.5: Unsupported platform: protoc-3.21.5-osx-aarch_64.exe -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :hudi-kafka-connect
```

可以看出是版本冲突的问题，解决方案就需要重新安装符合的protoc。
可以到 protobuf releases 下载对应的文件，这里我选择符合操作系统的 [protoc-24.3-osx-aarch_64.zip](https://github.com/protocolbuffers/protobuf/releases/download/v24.3/protoc-24.3-osx-aarch_64.zip) 文件。

下载以后直接解压即可，但是我们进入 bin目录下打开protoc的时候会显示无法验证开发者。可以打开系统偏好设置下的安全性与隐私，下面会提示是否允许protoc，我们选择允许既可以。

选择允许后会直接打开这个界面，可以看到我们安装成功了。

```bash
Usage: ./protoc [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
                              If not found in any of the these directories,
                              the --descriptor_set_in descriptors will be
                              checked for required proto file.
  --version                   Show version info and exit.
  -h, --help                  Show this text and exit.
  --encode=MESSAGE_TYPE       Read a text-format message of the given type
                              from standard input and write it in binary
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --deterministic_output      When using --encode, ensure map fields are
                              deterministically ordered. Note that this order
                              is not canonical, and changes across builds or
                              releases of protoc.
  --decode=MESSAGE_TYPE       Read a binary message of the given type from
                              standard input and write it in text format
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode_raw                Read an arbitrary protocol message from
                              standard input and write the raw tag/value
                              pairs in text format to standard output.  No
                              PROTO_FILES should be given when using this
                              flag.
  --descriptor_set_in=FILES   Specifies a delimited list of FILES
                              each containing a FileDescriptorSet (a
                              protocol buffer defined in descriptor.proto).
                              The FileDescriptor for each of the PROTO_FILES
                              provided will be loaded from these
                              FileDescriptorSets. If a FileDescriptor
                              appears multiple times, the first occurrence
                              will be used.
  -oFILE,                     Writes a FileDescriptorSet (a protocol buffer,
    --descriptor_set_out=FILE defined in descriptor.proto) containing all of
                              the input files to FILE.
  --include_imports           When using --descriptor_set_out, also include
                              all dependencies of the input files in the
                              set, so that the set is self-contained.
  --include_source_info       When using --descriptor_set_out, do not strip
                              SourceCodeInfo from the FileDescriptorProto.
                              This results in vastly larger descriptors that
                              include information about the original
                              location of each decl in the source file as
                              well as surrounding comments.
  --retain_options            When using --descriptor_set_out, do not strip
                              any options from the FileDescriptorProto.
                              This results in potentially larger descriptors
                              that include information about options that were
                              only meant to be useful during compilation.
  --dependency_out=FILE       Write a dependency output file in the format
                              expected by make. This writes the transitive
                              set of input file paths to FILE
  --error_format=FORMAT       Set the format in which to print errors.
                              FORMAT may be 'gcc' (the default) or 'msvs'
                              (Microsoft Visual Studio format).
  --fatal_warnings            Make warnings be fatal (similar to -Werr in
                              gcc). This flag will make protoc return
                              with a non-zero exit code if any warnings
                              are generated.
  --print_free_field_numbers  Print the free field numbers of the messages
                              defined in the given proto files. Extension ranges
                              are counted as occupied fields numbers.
  --enable_codegen_trace      Enables tracing which parts of protoc are
                              responsible for what codegen output. Not supported
                              by all backends or on all platforms.
  --plugin=EXECUTABLE         Specifies a plugin executable to use.
                              Normally, protoc searches the PATH for
                              plugins, but you may specify additional
                              executables not in the path using this flag.
                              Additionally, EXECUTABLE may be of the form
                              NAME=PATH, in which case the given plugin name
                              is mapped to the given executable even if
                              the executable's own name differs.
  --cpp_out=OUT_DIR           Generate C++ header and source.
  --csharp_out=OUT_DIR        Generate C# source file.
  --java_out=OUT_DIR          Generate Java source file.
  --kotlin_out=OUT_DIR        Generate Kotlin file.
  --objc_out=OUT_DIR          Generate Objective-C header and source.
  --php_out=OUT_DIR           Generate PHP source file.
  --pyi_out=OUT_DIR           Generate python pyi stub.
  --python_out=OUT_DIR        Generate Python source file.
  --ruby_out=OUT_DIR          Generate Ruby source file.
  --rust_out=OUT_DIR          Generate Rust sources.
  @<filename>                 Read options and filenames from file. If a
                              relative file path is specified, the file
                              will be searched in the working directory.
                              The --proto_path option will not affect how
                              this argument file is searched. Content of
                              the file will be expanded in the position of
                              @<filename> as in the argument list. Note
                              that shell expansion is not applied to the
                              content of the file (i.e., you cannot use
                              quotes, wildcards, escapes, commands, etc.).
                              Each line corresponds to a single argument,
                              even if it contains spaces.
```



然后再重新尝试打包Hudi项目，主要将hadoop、spark版本指定为cdh版本，重新编译：

```bash
mvn clean package -DskipTests -Dspark3.2 -Dflink1.14 -Dscala-2.12 -Drat.skip=true -Dhadoop.version=3.2.4
```

```bash
[INFO] Reactor Summary for Hudi 0.12.3:
[INFO] 
[INFO] Hudi ............................................... SUCCESS [  1.010 s]
[INFO] hudi-tests-common .................................. SUCCESS [  0.585 s]
[INFO] hudi-common ........................................ SUCCESS [  8.311 s]
[INFO] hudi-hadoop-mr ..................................... SUCCESS [  2.003 s]
[INFO] hudi-sync-common ................................... SUCCESS [  0.659 s]
[INFO] hudi-hive-sync ..................................... SUCCESS [  2.212 s]
[INFO] hudi-aws ........................................... SUCCESS [  1.127 s]
[INFO] hudi-timeline-service .............................. SUCCESS [  0.838 s]
[INFO] hudi-client ........................................ SUCCESS [  0.028 s]
[INFO] hudi-client-common ................................. SUCCESS [  5.412 s]
[INFO] hudi-spark-client .................................. SUCCESS [ 10.706 s]
[INFO] hudi-spark-datasource .............................. SUCCESS [  0.025 s]
[INFO] hudi-spark-common_2.12 ............................. SUCCESS [ 11.686 s]
[INFO] hudi-spark3-common ................................. SUCCESS [  4.374 s]
[INFO] hudi-spark3.2plus-common ........................... SUCCESS [  5.399 s]
[INFO] hudi-spark3.2.x_2.12 ............................... SUCCESS [  9.605 s]
[INFO] hudi-java-client ................................... SUCCESS [  1.307 s]
[INFO] hudi-spark_2.12 .................................... SUCCESS [ 18.723 s]
[INFO] hudi-utilities_2.12 ................................ SUCCESS [  3.523 s]
[INFO] hudi-utilities-bundle_2.12 ......................... SUCCESS [ 18.907 s]
[INFO] hudi-cli ........................................... SUCCESS [  8.284 s]
[INFO] hudi-flink-client .................................. SUCCESS [ 11.176 s]
[INFO] hudi-gcp ........................................... SUCCESS [  0.608 s]
[INFO] hudi-datahub-sync .................................. SUCCESS [  0.520 s]
[INFO] hudi-adb-sync ...................................... SUCCESS [  0.802 s]
[INFO] hudi-sync .......................................... SUCCESS [  0.024 s]
[INFO] hudi-hadoop-mr-bundle .............................. SUCCESS [  8.850 s]
[INFO] hudi-datahub-sync-bundle ........................... SUCCESS [ 10.978 s]
[INFO] hudi-hive-sync-bundle .............................. SUCCESS [  8.952 s]
[INFO] hudi-aws-bundle .................................... SUCCESS [ 10.142 s]
[INFO] hudi-gcp-bundle .................................... SUCCESS [  8.953 s]
[INFO] hudi-spark3.2-bundle_2.12 .......................... SUCCESS [ 17.265 s]
[INFO] hudi-presto-bundle ................................. SUCCESS [ 10.007 s]
[INFO] hudi-utilities-slim-bundle_2.12 .................... SUCCESS [ 15.409 s]
[INFO] hudi-timeline-server-bundle ........................ SUCCESS [ 13.625 s]
[INFO] hudi-trino-bundle .................................. SUCCESS [  9.932 s]
[INFO] hudi-examples ...................................... SUCCESS [  0.027 s]
[INFO] hudi-examples-common ............................... SUCCESS [  1.421 s]
[INFO] hudi-examples-spark ................................ SUCCESS [  4.631 s]
[INFO] hudi-flink-datasource .............................. SUCCESS [  0.025 s]
[INFO] hudi-flink1.14.x ................................... SUCCESS [  0.715 s]
[INFO] hudi-flink ......................................... SUCCESS [ 40.273 s]
[INFO] hudi-examples-flink ................................ SUCCESS [  0.945 s]
[INFO] hudi-examples-java ................................. SUCCESS [  1.715 s]
[INFO] hudi-flink1.13.x ................................... SUCCESS [  0.758 s]
[INFO] hudi-flink1.15.x ................................... SUCCESS [  0.770 s]
[INFO] hudi-kafka-connect ................................. SUCCESS [  3.661 s]
[INFO] hudi-flink1.14-bundle .............................. SUCCESS [ 16.274 s]
[INFO] hudi-kafka-connect-bundle .......................... SUCCESS [ 24.778 s]
[INFO] hudi-spark2_2.12 ................................... SUCCESS [  8.170 s]
[INFO] hudi-spark2-common ................................. SUCCESS [  0.034 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  05:46 min
[INFO] Finished at: 2023-09-14T11:54:44+08:00
[INFO] ------------------------------------------------------------------------
```

静静等待几分钟，可以发现打包成功了。