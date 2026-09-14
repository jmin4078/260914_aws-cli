export MY_SG_NAME="student13-web-sg"

export MY_SG_ID=$(aws ec2 describe-security-groups \
  --filters \
    "Name=group-name,Values=$MY_SG_NAME" \
    "Name=vpc-id,Values=$VPC_ID" \
  --query "SecurityGroups[0].GroupId" \
  --output text)

if [ "$MY_SG_ID" = "None" ] || [ -z "$MY_SG_ID" ]; then
  export MY_SG_ID=$(aws ec2 create-security-group \
    --group-name "$MY_SG_NAME" \
    --description "Security Group for Spring Boot and Nginx Practice" \
    --vpc-id "$VPC_ID" \
    --tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=$MY_SG_NAME},{Key=Course,Value=infra-training}]" \
    --query "GroupId" \
    --output text)
fi

echo "보안 그룹 ID: $MY_SG_ID"



export MY_KEY_NAME="student13-key-v2"
KEY_FILE="./$MY_KEY_NAME.pem"

if aws ec2 describe-key-pairs --key-names "$MY_KEY_NAME" >/dev/null 2>&1; then
  if [ -s "$KEY_FILE" ]; then
    echo "기존 키 페어와 로컬 PEM 파일을 사용합니다: $KEY_FILE"
  else
    echo "AWS에는 '$MY_KEY_NAME' 키가 있지만 사용할 수 있는 $KEY_FILE 파일이 없습니다." >&2
    echo "기존 개인 키는 다시 내려받을 수 없으므로 새 키 이름으로 생성해야 합니다." >&2
  fi
else
  if [ -e "$KEY_FILE" ]; then
    echo "기존 로컬 파일을 덮어쓰지 않았습니다: $KEY_FILE" >&2
    echo "MY_KEY_NAME을 새 이름으로 바꾼 뒤 다시 실행하세요." >&2
  else
    if aws ec2 create-key-pair \
      --key-name "$MY_KEY_NAME" \
      --query "KeyMaterial" \
      --output text > "$KEY_FILE"; then
      chmod 400 "$KEY_FILE"
      echo "키 페어 생성 완료: $KEY_FILE"
    else
      rm -f "$KEY_FILE"
      echo "키 페어 생성에 실패했습니다." >&2
    fi
  fi
fi

ls -l "$KEY_FILE"


if AMI_ID=$(aws ec2 describe-images \
  --owners 099720109477 \
  --filters \
    "Name=name,Values=ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-arm64-server-*" \
    "Name=state,Values=available" \
  --query "sort_by(Images,&CreationDate)[-1].ImageId" \
  --output text) && [ -n "$AMI_ID" ] && [ "$AMI_ID" != "None" ]; then
  export AMI_ID
  echo "최신 Ubuntu 24.04 ARM64 AMI: $AMI_ID"
else
  unset AMI_ID
  echo "Ubuntu 24.04 ARM64 AMI를 조회하지 못했습니다. AWS 리전을 확인하세요." >&2
fi


export MY_KEY_NAME="student13-key-3"

aws ec2 create-key-pair \
--key-name "$MY_KEY_NAME" \
--query "KeyMaterial" \
--output text > ./"$MY_KEY_NAME".pem

ls -l *.pem

export MY_SG_NAME="student13-web-sg"
export MY_SG_ID=$(aws ec2 create-security-group \
--group-name "$MY_SG_NAME" \
--description "Security Group for Spring Boot and Nginx Practice" \
--vpc-id "$VPC_ID" \
--tag-specifications "ResourceType=security-group,Tags=[{Key=Name,Value=$MY_SG_NAME},{Key=Course,Value=infra-training}]" \
--query "GroupId" --output text)
echo "생성된 보안 그룹 ID:$MY_SG_ID"