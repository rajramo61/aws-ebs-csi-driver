# CreateVolume Parameters
There are several optional parameters that may be passed into `CreateVolumeRequest.parameters` map, these parameters can be configured in StorageClass, see [example](../examples/kubernetes/storageclass).

The AWS EBS CSI Driver supports [tagging](tagging.md) through `StorageClass.parameters` (in v1.6.0 and later). 

| Parameters                  | Values                                                         | Default  | Description         |
|-----------------------------|----------------------------------------------------------------|----------|---------------------|
| "csi.storage.k8s.io/fstype" | xfs, ext2, ext3, ext4                                          | ext4     | File system type that will be formatted during volume creation. This parameter is case sensitive! |
| "type"                      | io1, io2, gp2, gp3, sc1, st1, standard, sbp1, sbg1  | gp3*     | EBS volume type.     |
| "iopsPerGB"                 |                                                                |          | I/O operations per second per GiB. Can be specified for IO1, IO2, and GP3 volumes. |
| "allowAutoIOPSPerGBIncrease"| true, false                                                    | false    | When `"true"`, the CSI driver increases IOPS for a volume when `iopsPerGB * <volume size>` is too low to fit into IOPS range supported by AWS. This allows dynamic provisioning to always succeed, even when user specifies too small PVC capacity or `iopsPerGB` value. On the other hand, it may introduce additional costs, as such volumes have higher IOPS than requested in `iopsPerGB`.|
| "iops"                      |                                                                |          | I/O operations per second. Can be specified for IO1, IO2, and GP3 volumes. |
| "throughput"                |                                                                | 125      | Throughput in MiB/s. Only effective when gp3 volume type is specified. If empty, it will set to 125MiB/s as documented [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html). |
| "encrypted"                 |                                                                |          | Whether the volume should be encrypted or not. Valid values are "true" or "false". |
| "kmsKeyId"                  |                                                                |          | The full ARN of the key to use when encrypting the volume. If not specified, AWS will use the default KMS key for the region the volume is in. This will be an auto-generated key called `/aws/ebs` if not changed. |

**Appendix**
* `gp3` is currently not supported on outposts. Outpost customers need to use a different type for their volumes.
* Unless explicitly noted, all parameters are case insensitive (e.g. "kmsKeyId", "kmskeyid" and any other combination of upper/lowercase characters can be used).
* If the requested IOPs (either directly from `iops` or from `iopsPerGB` multiplied by the volume's capacity) produces a value above the maximum IOPs allowed for the [volume type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html), the IOPS will be capped at the maximum value allowed. If the value is lower than the minimal supported IOPS value per volume, either an error is returned (the default behavior), or the value is increased to fit into the supported range when `allowautoiopspergbincrease` is `"true"`.
* You may specify either the "iops" or "iopsPerGb" parameters, not both. Specifying both parameters will result in an invalid StorageClass.

| Volume Type                 | Min total IOPS                         | Max total IOPS   | Max IOPS per GB   |
|-----------------------------|----------------------------------------|------------------|-------------------|
| IO1                         | 100                                    | 64000            | 50                |
| IO2                         | 100                                    | 64000            | 500               |
| GP3                         | 3000                                   | 16000            | 500               |