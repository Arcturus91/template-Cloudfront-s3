# CloudFront Distribution with S3 Origin

This CloudFormation template sets up a CloudFront distribution to serve content from a private S3 bucket.

## Template Details

- **AWSTemplateFormatVersion**: 2010-09-09
- **Description**: Distribución CloudFront para distribuir contenido de un bucket s3 privado

## Requerimientos:

1. Tener una cuenta AWS con permisos para desplegar stacks de CloudFormation.
2. Tener un bucket s3 configurado para ser privado, sin acceso público. Verifica que el contenido no se puede ver a menos que se entregue en un presigned URL.

## Usage

1. Actualiza el parámetro `S3BucketName` con el nombre de tu bucket S3 a exponer. Puede ser en la plantilla o directo en el la consola del Cloudformation
2. Despliega el stack desde tu terminal o desde la consola de Cloudformation en AWS.
3. Una vez que el stack esté desplegado, la distribución de CloudFront se creará y configurará para servir contenido desde el bucket S3 especificado.

## Parameters

- **S3BucketName**: 
  - **Type**: String
  - **Description**: Nombre del bucket S3 que será el origen de CloudFront

## Resources

- **CloudFrontOAI**: 
  - **Type**: AWS::CloudFront::CloudFrontOriginAccessIdentity
  - **Properties**: 
    - **CloudFrontOriginAccessIdentityConfig**: 
      - **Comment**: OAI para acceder a un bucket S3 privado

- **BucketPolicy**: 
  - **Type**: AWS::S3::BucketPolicy
  - **Properties**: 
    - **Bucket**: !Ref S3BucketName
    - **PolicyDocument**: 
      - **Version**: 2012-10-17
      - **Statement**: 
        - **Effect**: Allow
        - **Principal**: 
          - **CanonicalUser**: !GetAtt CloudFrontOAI.S3CanonicalUserId
        - **Action**: s3:GetObject
        - **Resource**: !Sub 'arn:aws:s3:::${S3BucketName}/*'

- **CloudFrontDistribution**: 
  - **Type**: AWS::CloudFront::Distribution
  - **Properties**: 
    - **DistributionConfig**: 
      - **Enabled**: true
      - **PriceClass**: PriceClass_All
      - **Origins**: 
        - **DomainName**: !Sub '${S3BucketName}.s3.${AWS::Region}.amazonaws.com'
        - **Id**: S3Origin
        - **S3OriginConfig**: 
          - **OriginAccessIdentity**: !Sub 'origin-access-identity/cloudfront/${CloudFrontOAI}'
      - **DefaultCacheBehavior**: 
        - **TargetOriginId**: S3Origin
        - **ViewerProtocolPolicy**: redirect-to-https
        - **CachePolicyId**: 658327ea-f89d-4fab-a63d-7e88639e58f6
        - **AllowedMethods**: 
          - GET
          - HEAD
          - OPTIONS
        - **CachedMethods**: 
          - GET
          - HEAD
          - OPTIONS
      - **HttpVersion**: http2
      - **IPV6Enabled**: true

## Outputs

- **DistributionId**: 
  - **Description**: ID de la distribución CloudFront
  - **Value**: !Ref CloudFrontDistribution

- **DistributionDomainName**: 
  - **Description**: Dominio de la distribución CloudFront
  - **Value**: !GetAtt CloudFrontDistribution.DomainName

