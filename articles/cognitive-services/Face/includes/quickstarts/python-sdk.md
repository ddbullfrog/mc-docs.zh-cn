---
title: 人脸 Python 客户端库快速入门
description: 使用适用于 Python 的人脸客户端库来检测人脸、查找相似的人脸（按图像进行人脸搜索）并识别人脸（人脸识别搜索）。
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: cf056d58a84cfdcd752f38cf06af7b73b9326d94
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105458"
---
开始使用适用于 Python 的人脸客户端库进行人脸识别。 请按照以下步骤安装程序包并试用基本任务的示例代码。 通过人脸服务，可以访问用于检测和识别图像中的人脸的高级算法。

使用适用于 Python 的人脸客户端库可以：

* 在图像中检测人脸
* 查找相似人脸
* 创建和训练人员组
* 识别人脸
* 验证人脸

[参考文档](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/?view=azure-python) | [库源代码](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-face) | [包 (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-face/) | [示例](https://docs.microsoft.com/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>先决条件

* [Python 3.x](https://www.python.org/)
* Azure 订阅 - [免费创建订阅](https://www.azure.cn/pricing/details/cognitive-services/)
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.cn/#create/Microsoft.CognitiveServicesFace"  title="创建人脸资源"  target="_blank">创建人脸资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到人脸 API。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
* 获取密钥和终结点后，请为该密钥和终结点[创建环境变量](/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication)，分别命名为 `FACE_SUBSCRIPTION_KEY` 和 `FACE_ENDPOINT`。

## <a name="setting-up"></a>设置
 
### <a name="create-a-new-python-application"></a>创建新的 Python 应用程序

创建新的 Python 脚本 &mdash; 例如 *quickstart-file.py* 。 在喜好的编辑器或 IDE 中打开该文件，并导入以下库。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

然后，为该资源的 Azure 终结点和密钥创建变量。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

> [!NOTE]
> 如果在启动应用程序后创建了环境变量，则需要关闭再重新打开运行该应用程序的编辑器、IDE 或 shell 才能访问该变量。

### <a name="install-the-client-library"></a>安装客户端库

可使用以下方式安装客户端库：

```console
pip install --upgrade azure-cognitiveservices-vision-face
```

## <a name="object-model"></a>对象模型

以下类和接口将处理人脸 Python 客户端库的某些主要功能。

|名称|说明|
|---|---|
|[FaceClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient?view=azure-python) | 此类代表使用人脸服务的授权，使用所有人脸功能时都需要用到它。 请使用你的订阅信息实例化此类，然后使用它来生成其他类的实例。 |
|[FaceOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python)|此类处理可对人脸执行的基本检测和识别任务。 |
|[DetectedFace](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface?view=azure-python)|此类代表已从图像中的单个人脸检测到的所有数据。 可以使用它来检索有关人脸的详细信息。|
|[FaceListOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.facelistoperations?view=azure-python)|此类管理云中存储的 **FaceList** 构造，这些构造存储各种不同的人脸。 |
|[PersonGroupPersonOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations?view=azure-python)| 此类管理云中存储的 **Person** 构造，这些构造存储属于单个人员的一组人脸。|
|[PersonGroupOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongroupoperations?view=azure-python)| 此类管理云中存储的 **PersonGroup** 构造，这些构造存储各种不同的 **Person** 对象。 |
|[ShapshotOperations](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.snapshotoperations?view=azure-python)|此类管理快照功能；可以使用它来暂时保存所有基于云的人脸数据，并将这些数据迁移到新的 Azure 订阅。 |

## <a name="code-examples"></a>代码示例

这些代码片段演示如何使用适用于 Python 的人脸客户端库执行以下任务：

* [对客户端进行身份验证](#authenticate-the-client)
* [检测图像中的人脸](#detect-faces-in-an-image)
* [查找相似人脸](#find-similar-faces)
* [创建和训练人员组](#create-and-train-a-person-group)
* [识别人脸](#identify-a-face)
* [验证人脸](#verify-faces)

## <a name="authenticate-the-client"></a>验证客户端

> [!NOTE]
> 本快速入门假设你已为人脸密钥[创建了名为 `FACE_SUBSCRIPTION_KEY` 的环境变量](../../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication)。

使用终结点和密钥实例化某个客户端。 使用密钥创建 [CognitiveServicesCredentials](https://docs.microsoft.com/python/api/msrest/msrest.authentication.cognitiveservicescredentials?view=azure-python) 对象，然后在终结点上使用该对象创建 [FaceClient](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient?view=azure-python) 对象。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

## <a name="detect-faces-in-an-image"></a>在图像中检测人脸

以下代码检测远程图像中的人脸。 它将检测到的人脸 ID 输出到控制台，并将其存储在程序内存中。 然后，它在包含多个人员的图像中检测人脸，并将其 ID 输出到控制台。 更改 [detect_with_url](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python#detect-with-url-url--return-face-id-true--return-face-landmarks-false--return-face-attributes-none--recognition-model--recognition-01---return-recognition-model-false--detection-model--detection-01---custom-headers-none--raw-false----operation-config-) 方法中的参数可以返回包含每个 [DetectedFace](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface?view=azure-python) 对象的不同信息。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

有关更多检测方案，请参阅 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py) 上的示例代码。

### <a name="display-and-frame-faces"></a>显示和定格人脸

下面的代码使用 DetectedFace.faceRectangle 属性将给定的图像输出到显示屏并在人脸周围绘制矩形。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

![一位年轻的妇女，其脸部周围绘制了一个红色矩形](../../images/face-rectangle-result.png)

## <a name="find-similar-faces"></a>查找相似人脸

以下代码采用检测到的单个人脸（源），并搜索其他一组人脸（目标），以找到匹配项（按图像进行人脸搜索）。 找到匹配项后，它会将匹配的人脸的 ID 输出到控制台。

### <a name="find-matches"></a>查找匹配项

首先，运行上一部分（[检测图像中的人脸](#detect-faces-in-an-image)）所示的代码，以保存对单个人脸的引用。 然后运行以下代码，以获取对图像组中多个人脸的引用。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

然后添加以下代码块，以查找该组中第一个人脸的实例。 若要了解如何修改此行为，请参阅 [find_similar](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations?view=azure-python#find-similar-face-id--face-list-id-none--large-face-list-id-none--face-ids-none--max-num-of-candidates-returned-20--mode--matchperson---custom-headers-none--raw-false----operation-config-) 方法。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

### <a name="print-matches"></a>输出匹配项

使用以下代码将匹配详细信息输出到控制台。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

## <a name="create-and-train-a-person-group"></a>创建和训练人员组

以下代码创建包含三个不同 **Person** 对象的 **PersonGroup** 。 它将每个 **Person** 与一组示例图像相关联，然后进行训练以便能够识别每个人。 

### <a name="create-persongroup"></a>创建 PersonGroup

若要逐步完成此方案，需将以下图像保存到项目的根目录： https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images 。

此图像组包含三组人脸图像，这些图像对应于三个不同的人。 该代码定义三个 **Person** 对象，并将其关联到以 `woman`、`man` 和 `child` 开头的图像文件。

设置图像后，在脚本的顶部为要创建的 **PersonGroup** 对象定义一个标签。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

然后将以下代码添加到脚本的底部。 此代码创建一个 **PersonGroup** 对象和三个 **Person** 对象。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

### <a name="assign-faces-to-persons"></a>将人脸添加到 Person

以下代码按图像前缀对图像排序、检测人脸，然后将人脸分配到每个 **Person** 对象。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

### <a name="train-persongroup"></a>训练 PersonGroup

分配人脸后，必须训练 **PersonGroup** ，使其能够识别与其每个 **Person** 对象关联的视觉特征。 以下代码调用异步 **train** 方法并轮询结果，然后将状态输出到控制台。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

## <a name="identify-a-face"></a>识别人脸

识别操作采用一个（或多个）人员的图像，并在图像中查找每个人脸的标识（人脸识别搜索）。 它将每个检测到的人脸与某个 **PersonGroup** （面部特征已知的不同 **Person** 对象的数据库）进行比较。

> [!IMPORTANT]
> 若要运行此示例，必须先运行[创建和训练人员组](#create-and-train-a-person-group)中的代码。

### <a name="get-a-test-image"></a>获取测试图像

以下代码在项目根目录中查找图像 _test-image-person-group.jpg_ ，并检测该图像中的人脸。 可以使用用于 **PersonGroup** 管理的图像查找此图像： https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images 。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

### <a name="identify-faces"></a>标识人脸

**identify** 方法采用检测到的人脸数组，并将其与 **PersonGroup** 进行比较。 如果检测到的某个人脸与某个人相匹配，则它会保存结果。 此代码将详细的匹配结果输出到控制台。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

## <a name="verify-faces"></a>验证人脸

验证操作采用某个人脸 ID 和其他人脸 ID 或 Person 对象，并确定它们是否属于同一个人。

以下代码检测两个源图像中的人脸，然后针对从目标图像检测到的人脸来验证它们。

### <a name="get-test-images"></a>获取测试图像

以下代码块声明将指向验证操作的源和目标图像的变量。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

### <a name="detect-faces-for-verification"></a>检测人脸进行验证

以下代码检测源和目标图像中的人脸并将其保存到变量中。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

### <a name="get-verification-results"></a>获取验证结果

以下代码将每个源图像与目标图像进行比较并打印出一条消息，指示它们是否属于同一个人。

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

## <a name="run-the-application"></a>运行应用程序

使用 `python` 命令从应用程序目录运行人脸识别应用。

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>清理资源

如果想要清理并删除认知服务订阅，可以删除资源或资源组。 删除资源组同时也会删除与之相关联的任何其他资源。

* [门户](/cognitive-services/cognitive-services-apis-create-account#clean-up-resources)
* [Azure CLI](/cognitive-services/cognitive-services-apis-create-account-cli#clean-up-resources)

如果你在本快速入门中创建了 **PersonGroup** 并想要删除它，请在脚本中运行以下代码：

```python
# <snippet_imports>
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
from urllib.parse import urlparse
from io import BytesIO
# To install this module, run:
# python -m pip install Pillow
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person
# </snippet_imports>

'''
Face Quickstart

Examples include:
    - Detect Faces: detects faces in an image.
    - Find Similar: finds a similar face in an image using ID from Detect Faces.
    - Verify: compares two images to check if they are the same person or not.
    - Person Group: creates a person group and uses it to identify faces in other images.
    - Large Person Group: similar to person group, but with different API calls to handle scale.
    - Face List: creates a list of single-faced images, then gets data from list.
    - Large Face List: creates a large list for single-faced images, trains it, then gets data.

Prerequisites:
    - Python 3+
    - Install Face SDK: pip install azure-cognitiveservices-vision-face
    - In your root folder, add all images downloaded from here:
      https://github.com/Azure-examples/cognitive-services-sample-data-files/tree/master/Face/images
How to run:
    - Run from command line or an IDE
    - If the Person Group or Large Person Group (or Face List / Large Face List) examples get
      interrupted after creation, be sure to delete your created person group (lists) from the API,
      as you cannot create a new one with the same name. Use 'Person group - List' to check them all,
      and 'Person Group - Delete' to remove one. The examples have a delete function in them, but at the end.
      Person Group API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
      Face List API: https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524d
References:
    - Documentation: /cognitive-services/face/
    - SDK: https://docs.microsoft.com/en-us/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face?view=azure-python
    - All Face APIs: /cognitive-services/face/APIReference
'''

# <snippet_subvars>
# This key will serve all examples in this document.
KEY = "<your subscription key>"

# This endpoint will be used in all examples in this quickstart.
ENDPOINT = "<your api endpoint>"
# </snippet_subvars>

# <snippet_verify_baseurl>
# Base url for the Verify and Facelist/Large Facelist operations
IMAGE_BASE_URL = 'https://csdx.blob.core.chinacloudapi.cn/resources/Face/Images/'
# </snippet_verify_baseurl>

# <snippet_persongroupvars>
# Used in the Person Group Operations and Delete Person Group examples.
# You can call list_person_groups to print a list of preexisting PersonGroups.
# SOURCE_PERSON_GROUP_ID should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Used for the Delete Person Group example.
TARGET_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)
# </snippet_persongroupvars>

'''
Authenticate
All examples use the same client.
'''
# <snippet_auth>
# Create an authenticated FaceClient.
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
# </snippet_auth>
'''
END - Authenticate
'''

'''
Detect faces 
Detect faces in two images (get ID), draw rectangle around a third image.
'''
print('-----------------------------')
print()
print('DETECT FACES')
print()
# <snippet_detect>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Display the detected face ID in the first single-face image.
# Face IDs are used for comparison to faces (their IDs) detected in other images.
print('Detected face ID from', single_image_name, ':')
for face in detected_faces: print (face.face_id)
print()

# Save this ID for use in Find Similar
first_image_face_ID = detected_faces[0].face_id
# </snippet_detect>

# <snippet_detectgroup>
# Detect the faces in an image that contains multiple faces
# Each detected face gets assigned a new ID
multi_face_image_url = "http://www.historyplace.com/kennedy/president-family-portrait-closeup.jpg"
multi_image_name = os.path.basename(multi_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces2 = face_client.face.detect_with_url(url=multi_face_image_url, detectionModel='detection_02')
# </snippet_detectgroup>

print('Detected face IDs from', multi_image_name, ':')
if not detected_faces2:
    raise Exception('No face detected from image {}.'.format(multi_image_name))
else:
    for face in detected_faces2:
        print(face.face_id)
print()

'''
Print image and draw rectangles around faces
'''
# <snippet_frame>
# Detect a face in an image that contains a single face
single_face_image_url = 'https://raw.githubusercontent.com/Microsoft/Cognitive-Face-Windows/master/Data/detection1.jpg'
single_image_name = os.path.basename(single_face_image_url)
# We use detection model 2 because we are not retrieving attributes.
detected_faces = face_client.face.detect_with_url(url=single_face_image_url, detectionModel='detection_02')
if not detected_faces:
    raise Exception('No face detected from image {}'.format(single_image_name))

# Convert width height to a point in a rectangle
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    
    return ((left, top), (right, bottom))


# Download the image from the url
response = requests.get(single_face_image_url)
img = Image.open(BytesIO(response.content))

# For each face returned use the face rectangle and draw a red box.
print('Drawing rectangle around face... see popup for results.')
draw = ImageDraw.Draw(img)
for face in detected_faces:
    draw.rectangle(getRectangle(face), outline='red')

# Display the image in the users default image browser.
img.show()
# </snippet_frame>

print()
'''
END - Detect faces
'''

'''
Find a similar face
This example uses detected faces in a group photo to find a similar face using a single-faced image as query.
'''
print('-----------------------------')
print()
print('FIND SIMILAR')
print()
# <snippet_findsimilar>
# Search through faces detected in group image for the single face from first image.
# First, create a list of the face IDs found in the second image.
second_image_face_IDs = list(map(lambda x: x.face_id, detected_faces2))
# Next, find similar face IDs like the one detected in the first image.
similar_faces = face_client.face.find_similar(face_id=first_image_face_ID, face_ids=second_image_face_IDs)
if not similar_faces[0]:
    print('No similar faces found in', multi_image_name, '.')
# </snippet_findsimilar>

# <snippet_findsimilar_print>
# Print the details of the similar faces detected
print('Similar faces found in', multi_image_name + ':')
for face in similar_faces:
    first_image_face_ID = face.face_id
    # The similar face IDs of the single face image and the group image do not need to match, 
    # they are only used for identification purposes in each image.
    # The similar faces are matched using the Cognitive Services algorithm in find_similar().
    face_info = next(x for x in detected_faces2 if x.face_id == first_image_face_ID)
    if face_info:
        print('  Face ID: ', first_image_face_ID)
        print('  Face rectangle:')
        print('    Left: ', str(face_info.face_rectangle.left))
        print('    Top: ', str(face_info.face_rectangle.top))
        print('    Width: ', str(face_info.face_rectangle.width))
        print('    Height: ', str(face_info.face_rectangle.height))
# </snippet_findsimilar_print>
print()
'''
END - Find Similar
'''

'''
Verify
The Verify operation takes a face ID from DetectedFace or PersistedFace and either another face ID or a Person object 
and determines whether they belong to the same person. If you pass in a Person object, you can optionally pass in a 
PersonGroup to which that Person belongs to improve performance.
'''
print('-----------------------------')
print()
print('VERIFY')
print()
# <snippet_verify_photos>
# Create a list to hold the target photos of the same person
target_image_file_names = ['Family1-Dad1.jpg', 'Family1-Dad2.jpg']
# The source photos contain this person
source_image_file_name1 = 'Family1-Dad3.jpg'
source_image_file_name2 = 'Family1-Son1.jpg'
# </snippet_verify_photos>

# <snippet_verify_detect>
# Detect face(s) from source image 1, returns a list[DetectedFaces]
# We use detection model 2 because we are not retrieving attributes.
detected_faces1 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name1, detectionModel='detection_02')
# Add the returned face's face ID
source_image1_id = detected_faces1[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces1), source_image_file_name1))

# Detect face(s) from source image 2, returns a list[DetectedFaces]
detected_faces2 = face_client.face.detect_with_url(IMAGE_BASE_URL + source_image_file_name2, detectionModel='detection_02')
# Add the returned face's face ID
source_image2_id = detected_faces2[0].face_id
print('{} face(s) detected from image {}.'.format(len(detected_faces2), source_image_file_name2))

# List for the target face IDs (uuids)
detected_faces_ids = []
# Detect faces from target image url list, returns a list[DetectedFaces]
for image_file_name in target_image_file_names:
    # We use detection model 2 because we are not retrieving attributes.
    detected_faces = face_client.face.detect_with_url(IMAGE_BASE_URL + image_file_name, detectionModel='detection_02')
    # Add the returned face's face ID
    detected_faces_ids.append(detected_faces[0].face_id)
    print('{} face(s) detected from image {}.'.format(len(detected_faces), image_file_name))
# </snippet_verify_detect>

# <snippet_verify>
# Verification example for faces of the same person. The higher the confidence, the more identical the faces in the images are.
# Since target faces are the same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_same = face_client.face.verify_face_to_face(source_image1_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence)
    if verify_result_same.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name1, target_image_file_names[0], verify_result_same.confidence))

# Verification example for faces of different persons.
# Since target faces are same person, in this example, we can use the 1st ID in the detected_faces_ids list to compare.
verify_result_diff = face_client.face.verify_face_to_face(source_image2_id, detected_faces_ids[0])
print('Faces from {} & {} are of the same person, with confidence: {}'
    .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence)
    if verify_result_diff.is_identical
    else 'Faces from {} & {} are of a different person, with confidence: {}'
        .format(source_image_file_name2, target_image_file_names[0], verify_result_diff.confidence))
# </snippet_verify>
print()
'''
END - VERIFY
'''

'''
Create/Train/Detect/Identify Person Group
This example creates a Person Group, then trains it. It can then be used to detect and identify faces in other group images.
'''
print('-----------------------------')
print()
print('PERSON GROUP OPERATIONS')
print()

# <snippet_persongroup_create>
'''
Create the PersonGroup
'''
# Create empty Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
print('Person group:', PERSON_GROUP_ID)
face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name=PERSON_GROUP_ID)

# Define woman friend
woman = face_client.person_group_person.create(PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.person_group_person.create(PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.person_group_person.create(PERSON_GROUP_ID, "Child")
# </snippet_persongroup_create>

# <snippet_persongroup_assign>
'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, child.person_id, ch)
# </snippet_persongroup_assign>

# <snippet_persongroup_train>
'''
Train PersonGroup
'''
print()
print('Training the person group...')
# Train the person group
face_client.person_group.train(PERSON_GROUP_ID)

while (True):
    training_status = face_client.person_group.get_training_status(PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the person group has failed.')
    time.sleep(5)
# </snippet_persongroup_train>

# <snippet_identify_testimage>
'''
Identify a face against a defined PersonGroup
'''
# Group image for testing against
test_image_array = glob.glob('test-image-person-group.jpg')
image = open(test_image_array[0], 'r+b')

print('Pausing for 60 seconds to avoid triggering rate limit on Trial...')
time.sleep (60)

# Detect faces
face_ids = []
# We use detection model 2 because we are not retrieving attributes.
faces = face_client.face.detect_with_stream(image, detectionModel='detection_02')
for face in faces:
    face_ids.append(face.face_id)
# </snippet_identify_testimage>

# <snippet_identify>
# Identify faces
results = face_client.face.identify(face_ids, PERSON_GROUP_ID)
print('Identifying faces in {}'.format(os.path.basename(image.name)))
if not results:
    print('No person identified in the person group for faces from {}.'.format(os.path.basename(image.name)))
for person in results:
    if len(person.candidates) > 0:
        print('Person for face ID {} is identified in {} with a confidence of {}.'.format(person.face_id, os.path.basename(image.name), person.candidates[0].confidence)) # Get topmost confidence score
    else:
        print('No person identified for face ID {} in {}.'.format(person.face_id, os.path.basename(image.name)))
# </snippet_identify>
print()
'''
END - Create/Train/Detect/Identify Person Group
'''

'''
Create/List/Delete Large Person Group
Uses the same list used for creating a regular-sized person group.
The operations are similar in structure as the Person Group example.
'''
print('-----------------------------')
print()
print('LARGE PERSON GROUP OPERATIONS')
print()

# Large Person Group ID, should be all lowercase and alphanumeric. For example, 'mygroupname' (dashes are OK).
LARGE_PERSON_GROUP_ID = str(uuid.uuid4()) # assign a random ID (or name it anything)

# Create empty Large Person Group. Person Group ID must be lower case, alphanumeric, and/or with '-', '_'.
# The name and the ID can be either the same or different
print('Large person group:', LARGE_PERSON_GROUP_ID)
face_client.large_person_group.create(large_person_group_id=LARGE_PERSON_GROUP_ID, name=LARGE_PERSON_GROUP_ID)

# Define woman friend , by creating a large person group person
woman = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Woman")
# Define man friend
man = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Man")
# Define child friend
child = face_client.large_person_group_person.create(LARGE_PERSON_GROUP_ID, "Child")

'''
Detect faces and register to correct person
'''
# Find all jpeg images of friends in working directory
woman_images = [file for file in glob.glob('*.jpg') if file.startswith("w")]
man_images = [file for file in glob.glob('*.jpg') if file.startswith("m")]
child_images = [file for file in glob.glob('*.jpg') if file.startswith("ch")]

# Add to a woman person
for image in woman_images:
    w = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, woman.person_id, w)

# Add to a man person
for image in man_images:
    m = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, man.person_id, m)

# Add to a child person
for image in child_images:
    ch = open(image, 'r+b')
    face_client.large_person_group_person.add_face_from_stream(LARGE_PERSON_GROUP_ID, child.person_id, ch)

'''
Train LargePersonGroup
'''
print()
print('Training the large person group...')
# Train the person group
face_client.large_person_group.train(LARGE_PERSON_GROUP_ID)
# Check training status
while (True):
    training_status = face_client.large_person_group.get_training_status(LARGE_PERSON_GROUP_ID)
    print("Training status: {}.".format(training_status.status))
    print()
    if (training_status.status is TrainingStatusType.succeeded):
        break
    elif (training_status.status is TrainingStatusType.failed):
        sys.exit('Training the large person group has failed.')
    time.sleep(5)

# Now that we have created and trained a large person group, we can retrieve data from it.
# Returns a list[Person] of how many Persons were created/defined in the large person group.
person_list_large = face_client.large_person_group_person.list(large_person_group_id=LARGE_PERSON_GROUP_ID, start='')

print('Persisted Face IDs from {} large person group persons:'.format(len(person_list_large)))
print()
for person_large in person_list_large:
    for persisted_face_id in person_large.persisted_face_ids:
        print('The person {} has an image with ID: {}'.format(person_large.name, persisted_face_id))
print()

# After testing, delete the large person group, LargePersonGroupPersons also get deleted.
face_client.large_person_group.delete(LARGE_PERSON_GROUP_ID)
print("Deleted the large person group.")
print()
'''
END - Create/List/Delete Large Person Group
'''

'''
FACELIST
This example adds single-faced images from URL to a list, then gets data from the list.
'''
print('-----------------------------')
print()
print('FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
face_list_id = "my-face-list"
print("Creating face list: {}...".format(face_list_id))
print()
face_client.face_list.create(face_list_id=face_list_id, name=face_list_id)

# Add each face in our array to the facelist
for image_file_name in image_file_names:
    face_client.face_list.add_face_from_url(
        face_list_id=face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Get persisted faces from the face list.
the_face_list = face_client.face_list.get(face_list_id)
if not the_face_list :
    raise Exception("No persisted face in face list {}.".format(face_list_id))

print('Persisted face ids of images in face list:')
print()
for persisted_face in the_face_list.persisted_faces:
    print(persisted_face.persisted_face_id)

# Delete the face list, so you can retest (recreate) the list with same name.
face_client.face_list.delete(face_list_id=face_list_id)
print()
print("Deleted the face list: {}.\n".format(face_list_id))
'''
END - FACELIST
'''

'''
LARGE FACELIST
This example adds single-faced images from URL to a large-capacity list, then gets data from the list.
This list could handle up to 1 million images.
'''
print('-----------------------------')
print()
print('LARGE FACELIST OPERATIONS')
print()

# Create our list of URL images
image_file_names_large = [
    "Family1-Dad1.jpg",
    "Family1-Daughter1.jpg",
    "Family1-Mom1.jpg",
    "Family1-Son1.jpg",
    "Family2-Lady1.jpg",
    "Family2-Man1.jpg",
    "Family3-Lady1.jpg",
    "Family3-Man1.jpg"
]

# Create an empty face list with an assigned ID.
large_face_list_id = "my-large-face-list"
print("Creating large face list: {}...".format(large_face_list_id))
print()
face_client.large_face_list.create(large_face_list_id=large_face_list_id, name=large_face_list_id)

# Add each face in our array to the large face list
# Returns a PersistedFace
for image_file_name in image_file_names_large:
    face_client.large_face_list.add_face_from_url(
        large_face_list_id=large_face_list_id,
        url=IMAGE_BASE_URL + image_file_name,
        user_data=image_file_name
    )

# Train the large list. Must train before getting any data from the list.
# Training is not required of the regular-sized facelist.
print("Train large face list {}".format(large_face_list_id))
print()
face_client.large_face_list.train(large_face_list_id=large_face_list_id)

# Get training status
while (True):
    training_status_list = face_client.large_face_list.get_training_status(large_face_list_id=large_face_list_id)
    print("Training status: {}.".format(training_status_list.status))
    if training_status_list.status is TrainingStatusType.failed:
        raise Exception("Training failed with message {}.".format(training_status_list.message))
    if (training_status_list.status is TrainingStatusType.succeeded):
        break
    time.sleep(5)

# Returns a list[PersistedFace]. Can retrieve data from each face.
large_face_list_faces = face_client.large_face_list.list_faces(large_face_list_id)
if not large_face_list_faces :
    raise Exception("No persisted face in face list {}.".format(large_face_list_id))

print('Face ids of images in large face list:')
print()
for large_face in large_face_list_faces:
    print(large_face.persisted_face_id)

# Delete the large face list, so you can retest (recreate) the list with same name.
face_client.large_face_list.delete(large_face_list_id=large_face_list_id)
print()
print("Deleted the large face list: {}.\n".format(large_face_list_id))
'''
END - LARGE FACELIST
'''

'''
Delete Person Group
For testing purposes, delete the person group made in the Person Group Operations.
List the person groups in your account through the online testing console to check:
https://dev.cognitive.azure.cn/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248
'''
print('-----------------------------')
print()
print('DELETE PERSON GROUP')
print()
# <snippet_deletegroup>
# Delete the main person group.
face_client.person_group.delete(person_group_id=PERSON_GROUP_ID)
print("Deleted the person group {} from the source location.".format(PERSON_GROUP_ID))
print()
# </snippet_deletegroup>

print()
print('-----------------------------')
print()
print('End of quickstart.')
```

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解如何使用适用于 Python 的人脸客户端库来执行基本人脸识别任务。 接下来，请在参考文档中详细了解该库。

> [!div class="nextstepaction"]
> [人脸 API 参考 (Python)](https://docs.microsoft.com/python/api/azure-cognitiveservices-vision-face/?view=azure-python)

* [什么是人脸服务？](../../overview.md)
* 可以在 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py) 上找到此示例的源代码。

