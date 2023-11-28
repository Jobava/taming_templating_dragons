```yaml
common:
  labels:
    app: microservice
  selector:
    matchLabels:
      app: microservice
  podSpec:
    containers:
    - name: microservice
      ports:
      - containerPort: 80
      resources:
        requests:
          cpu: "100m"
        limits:
          cpu: "200m"

deployments:
  - name: microservice-v1-0-0
    version: v1.0.0
    image: microservice:v1.0.0
  - name: microservice-v1-1-0
    version: v1.1.0
    image: microservice:v1.1.0

versionController:
  name: version-controller
  image: bitnami/kubectl:latest
  scriptConfigMap: deployment-monitor-script
```

and

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Data.Yaml
import Control.Monad (forM_)
import qualified Data.HashMap.Strict as HM
import Data.Text (Text)

-- Define the data types
data CommonConfig = CommonConfig
  { labels :: HM.HashMap Text Text
  , selector :: HM.HashMap Text (HM.HashMap Text Text)
  , podSpec :: PodSpec
  } deriving (Show)

data PodSpec = PodSpec
  { containers :: [Container]
  } deriving (Show)

data Container = Container
  { name :: Text
  , ports :: [Port]
  , resources :: Resources
  } deriving (Show)

data Port = Port
  { containerPort :: Int
  } deriving (Show)

data Resources = Resources
  { requests :: HM.HashMap Text Text
  , limits :: HM.HashMap Text Text
  } deriving (Show)

data Deployment = Deployment
  { name :: Text
  , version :: Text
  , image :: Text
  } deriving (Show)

data VersionController = VersionController
  { vcName :: Text
  , vcImage :: Text
  , scriptConfigMap :: Text
  } deriving (Show)

data Config = Config
  { common :: CommonConfig
  , deployments :: [Deployment]
  , versionController :: VersionController
  } deriving (Show)

instance FromJSON CommonConfig where
  -- Parse the CommonConfig from YAML
  -- ...

instance FromJSON PodSpec where
  -- Parse the PodSpec from YAML
  -- ...

instance FromJSON Container where
  -- Parse the Container from YAML
  -- ...

instance FromJSON Port where
  -- Parse the Port from YAML
  -- ...

instance FromJSON Resources where
  -- Parse the Resources from YAML
  -- ...

instance FromJSON Deployment where
  -- Parse the Deployment from YAML
  -- ...

instance FromJSON Version
...
```
