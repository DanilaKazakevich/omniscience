Редактируем подуль карпентера до 19.21 в терраформе и применяем, обязательно поставить параметр разрешить создавать роль. Обновляесяч до 32.10 , накатываем црм и потом мигрируем с provision profile 

Вот то что у нас Терраформом  создаётся 


```shell
module.eks_cluster.module.addon-karpenter[0].data.aws_availability_zones.available
module.eks_cluster.module.addon-karpenter[0].data.aws_ecrpublic_authorization_token.token
module.eks_cluster.module.addon-karpenter[0].data.aws_partition.current
module.eks_cluster.module.addon-karpenter[0].helm_release.karpenter   #ставим сам карпентер через хелм
module.eks_cluster.module.addon-karpenter[0].module.karpenter.data.aws_caller_identity.current 
module.eks_cluster.module.addon-karpenter[0].module.karpenter.data.aws_iam_policy_document.irsa[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.data.aws_iam_policy_document.irsa_assume_role[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.data.aws_iam_policy_document.queue[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.data.aws_partition.current
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_rule.this["health_event"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_rule.this["instance_rebalance"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_rule.this["instance_state_change"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_rule.this["spot_interupt"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_target.this["health_event"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_target.this["instance_rebalance"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_target.this["instance_state_change"]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_cloudwatch_event_target.this["spot_interupt"]

# Вот тут создаём важные компоненты, т.к. 
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_instance_profile.this[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_policy.irsa[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_role.irsa[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_role_policy_attachment.irsa[0]

# А зачем это??
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_sqs_queue.this[0]
module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_sqs_queue_policy.this[0]
```


TF_WORKSPACE=cloud terraform -chdir=terraform/cloud -var-file=cloud.tfvars state show ""


```shell
# module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_instance_profile.this[0]:
resource "aws_iam_instance_profile" "this" {
    arn         = "arn:aws:iam::965067289393:instance-profile/Karpenter-saritasa-cloud-eks-20231122134729406500000006"
    create_date = "2023-11-22T13:47:30Z"
    id          = "Karpenter-saritasa-cloud-eks-20231122134729406500000006"
    name        = "Karpenter-saritasa-cloud-eks-20231122134729406500000006"
    name_prefix = "Karpenter-saritasa-cloud-eks-"
    path        = "/"
    role        = "cloud-apps-v1-eks-node-group-20230923161208340400000003"
    tags        = {


# module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_policy.irsa[0]:
resource "aws_iam_policy" "irsa" {
    arn         = "arn:aws:iam::965067289393:policy/KarpenterIRSA-saritasa-cloud-eks-20231122134802681000000008"
    description = "Karpenter IAM role for service account"
    id          = "arn:aws:iam::965067289393:policy/KarpenterIRSA-saritasa-cloud-eks-20231122134802681000000008"
    name        = "KarpenterIRSA-saritasa-cloud-eks-20231122134802681000000008"
    name_prefix = "KarpenterIRSA-saritasa-cloud-eks-"
    path        = "/"
    policy      = jsonencode(
        {
            Statement = [
                {
                    Action   = [
                        "pricing:GetProducts",
                        "ec2:DescribeSubnets",
                        "ec2:DescribeSpotPriceHistory",
                        "ec2:DescribeSecurityGroups",
                        "ec2:DescribeLaunchTemplates",
                        "ec2:DescribeInstances",
                        "ec2:DescribeInstanceTypes",
                        "ec2:DescribeInstanceTypeOfferings",
                        "ec2:DescribeImages",
                        "ec2:DescribeAvailabilityZones",
                        "ec2:CreateTags",
                        "ec2:CreateLaunchTemplate",
                        "ec2:CreateFleet",
                    ]
                    Effect   = "Allow"
                    Resource = "*"
                    Sid      = ""
                },
                {
                    Action    = [
                        "ec2:TerminateInstances",
                        "ec2:DeleteLaunchTemplate",
                    ]
                    Condition = {
                        StringEquals = {
                            "ec2:ResourceTag/karpenter.sh/discovery" = "saritasa-cloud-eks"
                        }
                    }
                    Effect    = "Allow"
                    Resource  = "*"
                    Sid       = ""
                },
                {
                    Action    = "ec2:RunInstances"
                    Condition = {
                        StringEquals = {
                            "ec2:ResourceTag/karpenter.sh/discovery" = "saritasa-cloud-eks"
                        }
                    }
                    Effect    = "Allow"
                    Resource  = "arn:aws:ec2:*:965067289393:launch-template/*"
                    Sid       = ""
                },
                {
                    Action   = "ec2:RunInstances"
                    Effect   = "Allow"
                    Resource = [
                        "arn:aws:ec2:*::snapshot/*",
                        "arn:aws:ec2:*::image/*",
                        "arn:aws:ec2:*:965067289393:volume/*",
                        "arn:aws:ec2:*:965067289393:subnet/*",
                        "arn:aws:ec2:*:965067289393:spot-instances-request/*",
                        "arn:aws:ec2:*:965067289393:security-group/*",
                        "arn:aws:ec2:*:965067289393:network-interface/*",
                        "arn:aws:ec2:*:965067289393:instance/*",
                    ]
                    Sid      = ""
                },
                {
                    Action   = "ssm:GetParameter"
                    Effect   = "Allow"
                    Resource = "arn:aws:ssm:*:*:parameter/aws/service/*"
                    Sid      = ""
                },
                {
                    Action   = "eks:DescribeCluster"
                    Effect   = "Allow"
                    Resource = "arn:aws:eks:*:965067289393:cluster/saritasa-cloud-eks"
                    Sid      = ""
                },
                {
                    Action   = "iam:PassRole"
                    Effect   = "Allow"
                    Resource = "arn:aws:iam::965067289393:role/cloud-apps-v1-eks-node-group-20230923161208340400000003"
                    Sid      = ""
                },
                {
                    Action   = [
                        "sqs:ReceiveMessage",
                        "sqs:GetQueueUrl",
                        "sqs:GetQueueAttributes",
                        "sqs:DeleteMessage",
                    ]
                    Effect   = "Allow"
                    Resource = "arn:aws:sqs:us-west-2:965067289393:Karpenter-saritasa-cloud-eks"
                    Sid      = ""
                },
                {
                    Action   = [
                        "iam:TagInstanceProfile",
                        "iam:RemoveRoleFromInstanceProfile",
                        "iam:GetInstanceProfile",
                        "iam:DeleteInstanceProfile",
                        "iam:CreateInstanceProfile",
                        "iam:AddRoleToInstanceProfile",
                    ]
                    Effect   = "Allow"
                    Resource = "*"
                    Sid      = ""
                },
            ]
            Version   = "2012-10-17"
        }
    )
    policy_id   = "ANPA6BMT6GMY2GDFY36GZ"

# module.eks_cluster.module.addon-karpenter[0].module.karpenter.aws_iam_role.irsa[0]:
resource "aws_iam_role" "irsa" {
    arn                   = "arn:aws:iam::965067289393:role/KarpenterIRSA-saritasa-cloud-eks-20231122134729322000000005"
    assume_role_policy    = jsonencode(
        {
            Statement = [
                {
                    Action    = "sts:AssumeRoleWithWebIdentity"
                    Condition = {
                        StringEquals = {
                            "oidc.eks.us-west-2.amazonaws.com/id/8BC1F1A693378F3EB2FBD4B3EA9CB737:aud" = "sts.amazonaws.com"
                            "oidc.eks.us-west-2.amazonaws.com/id/8BC1F1A693378F3EB2FBD4B3EA9CB737:sub" = "system:serviceaccount:karpenter:karpenter"
                        }
                    }
                    Effect    = "Allow"
                    Principal = {
                        Federated = "arn:aws:iam::965067289393:oidc-provider/oidc.eks.us-west-2.amazonaws.com/id/8BC1F1A693378F3EB2FBD4B3EA9CB737"
                    }
                    Sid       = ""
                },
            ]
            Version   = "2012-10-17"
        }
    )
    create_date           = "2023-11-22T13:47:30Z"
    description           = "Karpenter IAM role for service account"
    force_detach_policies = true
    id                    = "KarpenterIRSA-saritasa-cloud-eks-20231122134729322000000005"
    managed_policy_arns   = [
        "arn:aws:iam::965067289393:policy/KarpenterIRSA-saritasa-cloud-eks-20231122134802681000000008",
    ]
    max_session_duration  = 3600
    name                  = "KarpenterIRSA-saritasa-cloud-eks-20231122134729322000000005"
    name_prefix           = "KarpenterIRSA-saritasa-cloud-eks-"
    path                  = "/"
    role_last_used        = [
        {
            last_used_date = "2024-05-17T15:28:29Z"
            region         = "us-west-2"
        },
    ]


```
	Action   = "iam:PassRole"
	Effect   = "Allow"
	Resource = "arn:aws:iam::965067289393:role/cloud-apps-v1-eks-node-group-20230923161208340400000003"
	Sid      = ""

Зачем IAM PassRole - да это просто говорит о том, когда карпентер создаёт инсантс под своей ролью, он может давать инстансу роль cloud-apps-v1-eks-node-group-2023092316120834040000000

Насчёт Policy из гайда - что через терраформ создается вроде совпадает с тем, что предлагают через github raw скачать и применить


`aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws`


`helm registry logout public.ecr.aws`

`docker logout`.


