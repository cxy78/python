# python哔哩哔哩爬取
哔哩哔哩的数据大部分都是动态的，所以第一步就是要先获取动态的url,我使用的是xpath方法

#encoding=utf-8

import requests
from lxml import etree
import re
def parse_page(url):

    headers={
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 '
                     '(KHTML, like Gecko) Chrome/72.0.3626.81 Safari/537.36 SE 2.X MetaSr 1.0'
    }

    response = requests.get(url,headers=headers)
    text = response.text
    html = etree.HTML(text)
    # 获取每个视频的url
    up_zhu_ids = html.xpath('//*[@id="all-list"]/div[1]/div[2]/ul/li/a/@href')
    for up_zhu_id in up_zhu_ids:
        # 动态url
        dynamic_url = 'https://api.bilibili.com/x/web-interface/view?bvid={}'
        # 法一： up_zhu_id = re.sub('//','https://',up_zhu_id)
        # 法二：
        up_url = 'https:'+up_zhu_id
        # #将 bvid 使用 / 将网址分隔开,但需 BV134411U77k? ，其位于第四个元素
        split_bid = re.split('/',up_url)[4]
        split_bid = re.split('\?',split_bid)[0]
        dynamic_url = dynamic_url.format(split_bid)
        print(dynamic_url)
        # 解析动态url
        dynamic_response = requests.get(dynamic_url)
        dynamic_text = dynamic_response.text
        print(dynamic_text)

        # 获取每个视频的主id
        up_zhu_id = re.findall(r'"mid":(.*?),',dynamic_text)
        print(up_zhu_id)

        # 获取每个视频的用户名
        username = re.findall(r'"name":(.*?),',dynamic_text)
        print(username)

        #爬取关注数

        up_zhu_id_response = requests.get(up_url,headers=headers)
        up_zhu_id_text = up_zhu_id_response.text
        up_follow_number = re.findall(r'<div class="btn-panel">.*?<i.*?>.*?<span>(.*?)</span>',up_zhu_id_text,re.DOTALL)
        print(up_follow_number)

        # 爬取视频名称
        video_name_response = requests.get(up_url,headers=headers)
        video_name_text = video_name_response.text
        html = etree.HTML(video_name_text)
        video_name = html.xpath('//*[@id="viewbox_report"]/h1/@title')
        print(video_name)

        # 发布时间
        video_published_time_response = requests.get(up_url,headers=headers)
        video_published_time_text = video_published_time_response.text
        video_published_time = re.findall(r'<div class="l-con">.*?<span.*?>.*?<span>(.*?)</span>',video_published_time_text,re.DOTALL)
        print(video_published_time)

        # 视频播放量
        video_playback_num = re.findall(r'"view":(.*?,)',dynamic_text)
        print(video_playback_num)

        # 弹幕数
        video_barrage_num = re.findall(r'"danmaku":(.*?,)',dynamic_text)
        print(video_barrage_num)

        # 点赞量
        video_like_num = re.findall(r'"like":(.*?,)',dynamic_text)
        print(video_like_num)

        # 投币量
        video_coin_num = re.findall(r'"coin":(.*?,)',dynamic_text)
        print(video_coin_num)

        # 收藏量
        video_favorite_num = re.findall(r'"favorite":(.*?,)',dynamic_text)
        print(video_favorite_num)

        # 转发量
        video_forward_num = re.findall(r'"share":(.*?,)',dynamic_text)
        print(video_forward_num)

        # 分区1
        category_1_response = requests.get(up_url,headers=headers)
        category_1_text = category_1_response.text
        category_1 = re.findall(r'<div class="video-data">.*?<a.*?>(.*?)</a>',category_1_text,re.DOTALL)[0]
        print(category_1)

        # 分区2
        category_2_response = requests.get(up_url,headers=headers)
        category_2_text = category_1_response.text
        category_2 = re.findall(r'<i class="van-icon-general_enter_s">.*?<a.*?>(.*?)</a>',category_2_text,re.DOTALL)
        print(category_2)
def main():

    #搜索词
    search_terms = ["简历","简历模板","面试","实习","找工作","笔试","职场"]

    for x in range(len(search_terms)):
        url = 'https://search.bilibili.com/all?keyword={}'.format(search_terms[x])
        parse_page(url)
        abc = url.format(search_terms[x])
        # parse_page(abc)




if __name__ == '__main__':
    main()
