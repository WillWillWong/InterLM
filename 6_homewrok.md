### Lagent & AgentLego 实战应用搭建

#### 环境配置
- 使用Conda创建新的环境，并安装PyTorch。

  ```text
  mkdir -p /root/agent
  studio-conda -t agent -o pytorch-2.1.2
  ```

- 通过Git克隆Lagent和AgentLego的源码，并进行安装。

  ```text
  cd /root/agent
  conda activate agent
  git clone https://gitee.com/internlm/lagent.git
  cd lagent && git checkout 581d9fb && pip install -e . && cd ..
  git clone https://gitee.com/internlm/agentlego.git
  cd agentlego && git checkout 7769e0d && pip install -e . && cd ..
  ```

- 安装LMDeploy等其他依赖库，为后续操作做准备。

  ```text
  conda activate agent
  pip install lmdeploy==0.3.0
  ```

#### Lagent Web Demo使用
- 启动LMDeploy的api_server，为Lagent Web Demo提供后端支持。

  ```text
  conda activate lmdeploy
  lmdeploy serve api_server /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-7b \
                              --server-name 127.0.0.1 \
                              --model-name internlm2-chat-7b \
                              --cache-max-entry-count 0.1
  ```

- 在新终端启动Lagent Web Demo，并进行本地网页转发。

  ```text
  conda activate agent
  cd /root/agent/lagent/examples
  streamlit run internlm2_agent_web_demo.py --server.address 127.0.0.1 --server.port 6006
  ```

  ```text
  ssh -CNg -L 6006:127.0.0.1:6006 -L 23333:127.0.0.1:23333 root@ssh.intern-ai.org.cn -p 你的 ssh 端口号
  ```

- 通过Web Demo进行实际应用测试，如搜索技术报告。

  

#### Lagent自定义工具开发
- 在Lagent中自定义工具的步骤，包括继承BaseAction类和实现run方法。

  

- 一个调用高德API的自定义工具，完成实时路径查询功能。

  ```python
  import json
  import os
  import requests
  from typing import Optional, Type
  
  from lagent.actions.base_action import BaseAction, tool_api
  from lagent.actions.parser import BaseParser, JsonParser
  from lagent.schema import ActionReturn, ActionStatusCode
  
  class MapQuery(BaseAction):
      """Map plugin for querying weather information."""
      
      def __init__(self,
                   key: Optional[str] = None,
                   description: Optional[dict] = None,
                   parser: Type[BaseParser] = JsonParser,
                   enable: bool = True) -> None:
          super().__init__(description, parser, enable)
          key = os.environ.get('MAP_API_KEY', key)
          if key is None:
              raise ValueError(
                  'Please set Map API key either in the environment'
                  'as MAP_API_KEY or pass it as `key`')
          self.key = key
          self.location_query_url = "https://restapi.amap.com/v3/geocode/geo?"
          self.distance_query_url = "https://restapi.amap.com/v3/distance?"
  
      @tool_api
      def run(self, query_origin: str,query_dest: str) -> ActionReturn:
          """一个地图查询API，可以返回query_orign 和 query_dest 两地之间的距离，以及驾车预计时间。
          
          Args:
              query_origin (:class:`str`): The origin name to query.
              destination_origin (:class:`str`): The destinaiton name to query.
          """
          tool_return = ActionReturn(type=self.name)
          status_code, response = self._search(query_origin, query_dest)
          if status_code == -1:
              tool_return.errmsg = response['info']
              tool_return.state = ActionStatusCode.HTTP_ERROR
          elif status_code == '1':
              parsed_res = self._parse_results(response)
              tool_return.result = [dict(type='text', content=str(parsed_res))]
              tool_return.state = ActionStatusCode.SUCCESS
          else:
              tool_return.errmsg = str(status_code)
              tool_return.state = ActionStatusCode.API_ERROR
          return tool_return
      
      def _parse_results(self, results: dict) -> str:
          """Parse the results from amap API.
          
          Args:
              results (dict): The distance content from amap API
                  in json format.
          
          Returns:
              str: The parsed distance results.
          """
          result = results[0]
          data = [
              f'距离: {result["distance"]}m',
              f'预计行驶时间: {result["duration"]}'
          ]
          return '\n'.join(data)
  
      def _search(self, query_origin: str, query_destination: str):
          # get geo_code
          try:
              origin_code_response = requests.get(
                  self.location_query_url + f'address={query_origin}'+ f'&key={self.key}'
              )
              destination_code_response = requests.get(
                  self.location_query_url + f'address={query_destination}'+ f'&key={self.key}'
              )
          except Exception as e:
              return -1, str(e)
          if origin_code_response.status_code != 200:
              return origin_code_response.status_code, origin_code_response.json()
          origin_code_response = origin_code_response.json()
          if destination_code_response.status_code != 200:
              return destination_code_response.status_code, destination_code_response.json()
          destination_code_response = destination_code_response.json()
          if origin_code_response['status'] != 0:
              origin_location = origin_code_response['geocodes'][0]['location']
          else:
              return 0, '未查询到起点位置'
          if destination_code_response['status'] != 0:
              destination_location = destination_code_response['geocodes'][0]['location']
          else:
              return 0, '未查询到终点位置'
          # get distance(influenced by the road conditions)
          try:
              distance_response = requests.get(
                  self.distance_query_url + f'origins={origin_location}' +f'&destination={destination_location}'+f'&key={self.key}'
              ).json()
          except Exception as e:
              return 0, str(e)
          return distance_response['status'], distance_response['results']
  ```

#### AgentLego工具包使用
- 使用AgentLego提供的多种工具API，如目标检测工具。

  ```bash
  cd /root/agent
  wget http://download.openmmlab.com/agentlego/road.jpg
  ```

- AgentLego WebUI的使用，包括配置Agent和工具，以及进行对话交互。

  ```text
  conda activate agent
  pip install openmim==0.3.9
  mim install mmdet==3.3.0
  ```

#### AgentLego自定义工具开发
- 基于AgentLego构建自定义工具的过程，包括继承BaseTool类和实现apply方法。

  ```text
  def llm_internlm2_lmdeploy(cfg):
      url = cfg['url'].strip()
      llm = LMDeployClient(
          model_name='internlm2-chat-7b',
          url=url,
          meta_template=INTERNLM2_META,
          top_p=0.8,
          top_k=100,
          temperature=cfg.get('temperature', 0.7),
          repetition_penalty=1.0,
          stop_words=['<|im_end|>'])
      return llm
  ```

- 一个调用MagicMaker API的图像生成工具。

  

#### 实战演练总结
- 实战演练，展示了Lagent和AgentLego在智能体应用搭建中的应用。

```python
import json
import requests

import numpy as np

from agentlego.types import Annotated, ImageIO, Info
from agentlego.utils import require
from .base import BaseTool


class MagicMakerImageGeneration(BaseTool):

    default_desc = ('This tool can call the api of magicmaker to '
                    'generate an image according to the given keywords.')

    styles_option = [
        'dongman',  # 动漫
        'guofeng',  # 国风
        'xieshi',   # 写实
        'youhua',   # 油画
        'manghe',   # 盲盒
    ]
    aspect_ratio_options = [
        '16:9', '4:3', '3:2', '1:1',
        '2:3', '3:4', '9:16'
    ]

    @require('opencv-python')
    def __init__(self,
                 style='guofeng',
                 aspect_ratio='4:3'):
        super().__init__()
        if style in self.styles_option:
            self.style = style
        else:
            raise ValueError(f'The style must be one of {self.styles_option}')
        
        if aspect_ratio in self.aspect_ratio_options:
            self.aspect_ratio = aspect_ratio
        else:
            raise ValueError(f'The aspect ratio must be one of {aspect_ratio}')

    def apply(self,
              keywords: Annotated[str,
                                  Info('A series of Chinese keywords separated by comma.')]
        ) -> ImageIO:
        import cv2
        response = requests.post(
            url='https://magicmaker.openxlab.org.cn/gw/edit-anything/api/v1/bff/sd/generate',
            data=json.dumps({
                "official": True,
                "prompt": keywords,
                "style": self.style,
                "poseT": False,
                "aspectRatio": self.aspect_ratio
            }),
            headers={'content-type': 'application/json'}
        )
        image_url = response.json()['data']['imgUrl']
        image_response = requests.get(image_url)
        image = cv2.imdecode(np.frombuffer(image_response.content, np.uint8), cv2.IMREAD_COLOR)
        return ImageIO(image)
```

接下来修改 `/root/AgentLego/agentlego/agentlego/tools/__init__.py `文件，将我们的工具注册在工具列表中。如下所示，我们将 `MagicMakerImageGeneration` 通过 `from .magicmaker_image_generation import MagicMakerImageGeneration` 导入到了文件中，并且将其加入了` __all__ `列表中。

```python
from .base import BaseTool
from .calculator import Calculator
from .func import make_tool
from .image_canny import CannyTextToImage, ImageToCanny
from .image_depth import DepthTextToImage, ImageToDepth
from .image_editing import ImageExpansion, ImageStylization, ObjectRemove, ObjectReplace
from .image_pose import HumanBodyPose, HumanFaceLandmark, PoseToImage
from .image_scribble import ImageToScribble, ScribbleTextToImage
from .image_text import ImageDescription, TextToImage
from .imagebind import AudioImageToImage, AudioTextToImage, AudioToImage, ThermalToImage
from .object_detection import ObjectDetection, TextToBbox
from .ocr import OCR
from .scholar import *  # noqa: F401, F403
from .search import BingSearch, GoogleSearch
from .segmentation import SegmentAnything, SegmentObject, SemanticSegmentation
from .speech_text import SpeechToText, TextToSpeech
from .translation import Translation
from .vqa import VQA
from .magicmaker_image_generation import MagicMakerImageGeneration

__all__ = [
    'CannyTextToImage', 'ImageToCanny', 'DepthTextToImage', 'ImageToDepth',
    'ImageExpansion', 'ObjectRemove', 'ObjectReplace', 'HumanFaceLandmark',
    'HumanBodyPose', 'PoseToImage', 'ImageToScribble', 'ScribbleTextToImage',
    'ImageDescription', 'TextToImage', 'VQA', 'ObjectDetection', 'TextToBbox', 'OCR',
    'SegmentObject', 'SegmentAnything', 'SemanticSegmentation', 'ImageStylization',
    'AudioToImage', 'ThermalToImage', 'AudioImageToImage', 'AudioTextToImage',
    'SpeechToText', 'TextToSpeech', 'Translation', 'GoogleSearch', 'Calculator',
    'BaseTool', 'make_tool', 'BingSearch', 'MagicMakerImageGeneration'
]
```

- 通过跟随教程，学习如何配置环境、自定义工具以及进行实战应用搭建。

