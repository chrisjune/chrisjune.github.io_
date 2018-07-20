---
layout: post-sidebar
date: 2018-07-07
title: "장고 - View"
categories: django
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 15
feature_image: feature-san-fran
show_related_posts: true
square_related: recommend-spain
---
`askdjango`를 참고하였습니다
### View
* urlconf에 매핑된 object
* 첫인자로 httpresquest를 받고,
* 반드시 httpresponse를 리턴해야한다.

### FBV(function based view)
* 엑셀다운로드 예제

```python
import os
from django.http import HttpResponse, JsonResponse
def post_list(request):
	return JsonResponse({
		'message': 'hi django',
		'items': ['python','dj','celer'],
	}, json_dumps_params={'ensure_ascii':False})
```

* 엑셀다운로드 예제2

```python
import os
from django.http import HttpResponse, JsonResponse
def excep_download(request):
	filepath='/User/usr/excel.xsl'
	filename=os.path.basename(filepath)
	
	with open(filepath, 'rb') as f:
		response = HttpResponse(f, conetent_type='application/vnd.ms-excel')
		response['Content-Disposition'] = 'attachment; filename="{}"'.format(filename)
		return response
```


### CBV(class based view)
* django.views.generic: 뷰사용패턴을 일반화 시켜놓은 뷰의 모음
* as_view(): 클래스를 통해 호출되는 함수를 통해 FBV(Function based view)를 생성해주는 클래스

* CBV 예제

```python
from django.views.generic import View
from django.http import HttpResponse
class PostListView1(View):
	def get(self, request):
		name = 'testname'
		html = self.get_template_string().format(name=name)
		return HttpResponse(html)
	def get_tempalte_string(self):
		return "<h1>title</h1>	<p>asdfasdf<p>"
```	


### 템플릿뷰
* 템플릿 뷰 예제

```python
class PostListView2(TemplateView):
	template_view='app/post_list.html'
	def get_context_data(self):
		context = super().get_context_data()
		context['name'] = 'testname'
		return context
```

* JsonResponse 템플릿뷰 예제

```python
from django.http import JsonResponse
from django.views.generic import View
class PostListView3(View):
	def get(self, request):
		return JsonResponse(self.get_data(), 		json_dumps_params={'ensure_ascii':False})
	
	def get_data(self):
	return {
		'message':'hi python',
		'items': ['python','dj','celery','azure','aws']
	}
post_list3 = PostListView3().as_view()
```

* 엑셀 다운로드 예제

```python
import os
from django.http import HttpResponse
from django.views.generic improt View

class ExcelDownlaodView(View):
	excep_path = 'User/usr/excel.xls'
	
	def get(self,request):
		filename = os.path.basename(self.excel_path)
		with open(filename,'rb') as f:
			reponse = HttpResponse(f, content_type = 'application/vnd.ms-excel')
			response['Content-Disposition'] = 'attachment; filename="{}"'.format(filename)
			return response
```			
