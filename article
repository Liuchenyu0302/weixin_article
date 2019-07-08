import requests
import json
import time
from pymongo import MongoClient

headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36'
    }
    
# Mongo配置
conn = MongoClient('127.0.0.1', 27017)
db = conn.wxgzh  # 连接wx数据库，没有则自动创建
mongo_wxgzh = db.article  # 使用article集合，没有则自动创建

class Gongzhonghao(object):
    """
    爬取公众号《smart晨》的所有文章
    """
    def __init__(self,index=0):
        #文章的url，设置为动态填充
        self.url = 'http://mp.weixin.qq.com/mp/profile_ext?action=getmsg&__biz=MzU5MzgzNzQ5NA==&f=json&offset={offset}&count=10&is_ok=1&scene=124&uin=MjAxNjM2MzQyMA%3D%3D&key=c057d35eab709d25a018580dc67431367c9a3143a844e56882dd973943fafe48a78e6ba7022257638ffa5c4368d00e88216c4189c10a4b0a7a11bf5e93c8ab375346b719750d54997e6453c1e076e25a&pass_ticket=Bo6lVc68c2qlxZYmRedyA2RqnJX7Emb96WqKozpPmboJbN2P8p1hcgyTP2s2NUTk&wxtoken=&appmsg_token=1016_ggLYJHXMRDPAo4NJ4srIf0h_8QwVao_bJ6qjOw~~&x5=0&f=json HTTP/1.1'

        self.offset = (index + 1) * 10

    def start_request(self):
        #返回完整的url
        # yield self.url.format(biz=self.biz,uin=self.uin,key=self.key,offset=self.offset,count=self.count,action=self.action,f=self.f)
        yield self.url.format(offset=self.offset)

    def run(self,index):
        #获取每个分页
        while true:
            print(f'开始抓取公众号第{index + 1} 页文章.')
            flag = self.get_article()
            # 下载间隔设置一下
            time.sleep(5)
            index += 1
            if not flag:
                print('公众号文章已全部抓取完毕，退出程序.')
                break
            print(f'准备抓取公众号第{index + 1} 页文章.')

    def get_article(self):
        #获取单个页面内所有文章的详细信息
        response = requests.get(self.url, headers=headers)
        resp_json = response.json()
        if resp_json.get('errmsg') == 'ok':
            resp_json = response.json()
            # 是否还有分页数据， 用于判断return的值
            can_msg_continue = resp_json['can_msg_continue']
            # 当前分页文章数
            msg_count = resp_json['msg_count']
            general_msg_list = json.loads(resp_json['general_msg_list'])
            list = general_msg_list.get('list')
            print(list)
            print("=" * 50)
            for i in list:
                try:
                    app_msg_ext_info = i['app_msg_ext_info']
                    # 标题
                    title = app_msg_ext_info['title']
                    # 文章地址
                    content_url = app_msg_ext_info['content_url']
                    # 封面图
                    cover = app_msg_ext_info['cover']

                    # 发布时间
                    datetime = i['comm_msg_info']['datetime']
                    datetime = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(datetime))

                    #将数据插入到MongoDB中
                    mongo_wxgzh.insert({
                        'title': title,
                        'content_url': content_url,
                        'cover': cover,
                        'datetime': datetime
                    })
                except:
                    pass
            if can_msg_continue == 1:
                return True
            return False
        else:
            print('获取文章异常...')
            return False


if __name__ == '__main__':
    gzh = Gongzhonghao()
    gzh.run(0)
