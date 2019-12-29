# Packer术语

- `Artifacts`（工件）

  一次构建的结果，通常是代表主机镜像的一组ID或文件。每个构建器都会生成一个`artifact`。例如，在EC2构建器中，`artifact`是一组AMI ID。对于VMware构建器，`artifact`是一个文件目录，其中包含创建的虚拟机。

  

