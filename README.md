Là một lập trình viên, chắc hẳn trong chúng ta, chưa ai là chưa nghe tới ngôn ngữ lập trình NodeJs, một trong những ngôn ngữ lập trình được sử dụng rộng rãi nhất hiện nay, bởi sự hiệu quả, tính ổn định của nó. Nodejs không ngừng phát triển, cùng với sự phát triển của ngôn ngữ lập trình, kéo theo đó là có vô số những framework ra đời để hỗ trợ những lập trình viên trong viện code và ứng dụng ngôn ngữ lập trình vào dự án. Một trong số đó là NestJs.

Giới thiệu về NestJs thì mình cũng có riêng một số bài viết về framework này, các bạn có thể tìm trên repository cá nhân của mình hoặc có thể tự google search để đọc, hiểu hơn về framework này nhé.

Trong bài viết này, mình sẽ xử dụng framework NestJs để xây dựng một ứng dụng REST API đơn giản (chức năng CRUD). Trong ví dụ này, mình sử dụng SQLite làm database bởi vì chúng ta không cần quá mất công vào việc cài đặt nó mà chức năng của nó thì tương tự như MySQL hay là Oracle. 

## Cài đặt NestJS cli và tạo mới project

Trước tiên, hãy cài NestJs cli để có thể create mới 1 project Nestjs. Mở terminal của bạn lên và gõ dòng lệnh

```
$ npm install -g @nest/cli
```

Sau khi hoàn tất câu lệnh trên, tạo mới một project bằng lênh

```
$ nest new crud-app
```

Với _crud-app_ là tên project, tất nhiên các bạn có thể đặt tên tùy ý theo ý thích của mình. Nhưng cũng lưu ý với mọi người là nên đặt tên project dễ đọc, dễ nhớ, và có thể hiểu được chức năng chính của project khi đọc tên. Sau khi tạo xong project mới, hãy di chuyển đường dẫn vào trong thư mục vừa tạo và khởi động server lên bằng lệnh

```
$ cd crud-app
$ npm run start:dev
```

Chờ tới khi câu lệnh run thành công, bạn hãy truy cập vào đường dẫn  [http://localhost:3000](http://localhost:3000) trên trình duyệt của bạn. Lúc này, bạn sẽ nhìn thấy trang welcome của nestjs với dòng chữ **Hello World!** trên màn hình. 

Việc chuẩn bị đã hoàn tất, bây giờ chúng ta sẽ đi vào công việc chính của bài viết này

## Tạo mới Module

Để cấu trúc dự án tốt nhất, code dự án của chúng ta, vừa gọn gàng, sạch sẽ, lại dễ đọc, dễ code thì chúng ta cần tạo một module chứa tất cả code của ứng dụng CRUD chúng ta muốn làm. Để tạo được module, mở terminal lên với root là thư mục project của bạn, để tạo một module với tên _contacts_ chúng ta sử dụng câu lệnh sau:

```
$ nest generate module contacts
```

Sau khi câu lệnh trên hoàn tất, chúng ta sẽ được một file mới với tên _src/contacts/contacts.module.ts_ với đoạn code default

```
import { Module } from '@nestjs/common';

@Module({})
export class ContactsModule {}
```

Và trong file _src/app.module.ts_ sẽ được includes contacts module như sau:

```
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ContactsModule } from './contacts/contacts.module';

@Module({
  imports: [ContactsModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Tiếp đến, chúng ta cần tạo một services để đóng gói tất cả các họat động CRUD bằng dòng lệnh

```
$ nest generate service contacts/contacts
```

Câu lệnh trên sẽ tạo ra file _src/contacts/contacts.service.spec.ts_ và file _src/contacts/contacts.service.ts_ đồng thời update file _src/contacts/contacts.module.ts_ để include thêm service vào trong contacts module.

Tiếp theo, hãy tạo controller contacts và tất nhiên controller này cần được tạo trong module contacts bằng câu lệnh

```
$ nest generate controller contacts/contacts
```

Với câu lệnh trên, chúng ta sẽ được file _src/contacts/contacts/contacts.controller.spec.ts_ và file _src/contacts/contacts/contacts.controller.ts_ đồng thời file _src/contacts/contacts.module.ts_ cũng sẽ được update

Mở file _src/contacts/contacts/contacts.controller.ts_ và thêm vào route:

```
import { Controller, Get } from '@nestjs/common';

@Controller('contacts')
export class ContactsController {
    @Get()
    index(): string {
      return "This action will return contacts";
    }    
}
```

Trước hết, chung ta import Get() decorator từ package _@nestjs/common_ và chúng ta dùng nó để decorate method index(), nó sẽ tạo route với path là _/contacts_. Method của chúng ta hiện tại chỉ trả về dòng chữ _This action will return contacts_

Một phần quan trong không kém nữa là khởi tạo database. Bây giờ chúng ta sẽ bắt tay vào làm điều đó. 

## Setting up Database

Trước hết, chúng ta cần cài thêm typeORM package vào project của mình bằng câu lệnh

```
$ npm install --save @nestjs/typeorm typeorm sqlite3
```

Tiếp theo, cần phải khai báo TypeORMModule, mở file _src/app.module.ts_ và thêm đoạn code sau

```
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ContactsModule } from './contacts/contacts.module';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [ContactsModule,       
  TypeOrmModule.forRoot({
    type: 'sqlite',
    database: 'db',
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    synchronize: true,
 }),],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Chúng ta sử dụng method forRoot() để truyền vào config của typeORM

## Tạo Entity Model

Tạo mới file entity bằng lệnh

```
$ touch src/contacts/contact.entity.ts
```
Mở file vừa được tạo vào add vào dòng code

```
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Contact {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    firstName: string;

    @Column()
    lastName: string;

    @Column()
    email: string;

    @Column()
    phone: string;

    @Column()
    city: string;

    @Column()
    country: string;
}
```

Tiếp theo, mở file _src/contacts/contacts.module.ts_ và update nội dung file này với code sau:

```
import { Module } from '@nestjs/common';
import { ContactsService } from './contacts.service';
import { ContactsController } from './contacts/contacts.controller';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Contact } from './contact.entity';

@Module({
  imports: [
    TypeOrmModule.forFeature([Contact]),
  ],
  providers: [ContactsService],
  controllers: [ContactsController]
})
export class ContactsModule {}
```

Những nội dung mới đó là

- TypeOrmModule được import vào từ '@nestjs/typeorm'
- Entity được import vào với tên Contact từ './contact.entity'
- TypeOrmModule.forFeature([Contact]) được gọi trong imports array

Bây giờ TypeORM sẽ nhận diện được Contact entity và đồng bộ cơ sở dữ liệu tương ứng bằng cách tạo bảng quan hệ. Nếu muốn xác thực bạn có thẻ xử dụng SQLite database brower.

## Tạo CRUD Service

Mở file _src/contacts/contacts.service.ts_ và thêm vào đoạn code sau:

```
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Contact } from './contact.entity';

@Injectable()
export class ContactsService {
    constructor(
        @InjectRepository(Contact)
        private contactRepository: Repository<Contact>,
    ) { }
}
```

Chúng ta đã import Contact entity, Repository, và InjectRepository. Tiếp theo, chúng ta import Contact entity từ contructor của service. contactRepository cung cấp cho chúng ta những method để chạy CRUD tới database. Tiếp tới, chúng ta cần define những hàm CRUD. Vẫn trong file service này, import thêm

```
import { UpdateResult, DeleteResult } from  'typeorm';
```

Tiếp theo, định nghĩ những method mà chúng ta cần:

```
 async  findAll(): Promise<Contact[]> {
        return await this.contactRepository.find();
    }

    async  create(contact: Contact): Promise<Contact> {
        return await this.contactRepository.save(contact);
    }

    async update(contact: Contact): Promise<UpdateResult> {
        return await this.contactRepository.update(contact.id, contact);
    }

    async delete(id): Promise<DeleteResult> {
        return await this.contactRepository.delete(id);
    }
```

Quay lại file _src/contacts/contacts/contacts.controller.ts_ và update file này như sau.

Trước hết, cần import service và entity contact chúng ta thao tác bên trên vào đã

```
import { Contact } from '../contact.entity';
import { ContactsService } from '../contacts.service';
```

Tiếp tới, inject ContactsService vào thông qua contructor của controller

```
export class ContactsController {
    constructor(private contactsService: ContactsService){}
```

Cuối cùng update method index() với đonạ code sau:

```
@Controller('contacts')
export class ContactsController {
    constructor(private contactsService: ContactsService){}

    @Get()
    index(): Promise<Contact[]> {
      return this.contactsService.findAll();
    }    
}
```

Nếu bạn reload trình duyệt [localhost:3000/contacts](localhost:3000/contacts) thì sẽ thấy kết quả trả về là một mảng rỗng, cũng dễ hiểu thôi, vì trong DB chúng ta chưa có bát kỳ một record nào mà. =))

Việt tiếp theo sẽ là insert dữ liệu vào DB. Tương tự như trên, trước hết chúng ta cũng cần import package vào 

```
import { Post,Put, Delete, Body, Param } from  '@nestjs/common';
```
Tiếp theo, định nghĩa route mới bằng đoạn code:

```
 @Post('create')
    async create(@Body() contactData: Contact): Promise<any> {
      return this.contactsService.create(contactData);
    }  
```

Chúng ta dùng method POST để tạo request post. đồng thời, với việc khai bao trên, thì cũng sinh ra cho chúng ta route mới là _/contacts/create_. parmam truyền lên request là contactData. Tiếp theo, khai báo method cho việc update dữ liệu

```
@Put(':id/update')
    async update(@Param('id') id, @Body() contactData: Contact): Promise<any> {
        contactData.id = Number(id);
        console.log('Update #' + contactData.id)
        return this.contactsService.update(contactData);
    }  
```

Method update, thì parame chúng ta cần truyền vào là id của item muốn update. Cuối cùng là method delete, tương tự update thì cũng cần truền vào id của item chúng ta muốn xóa. 

```
@Delete(':id/delete')
    async delete(@Param('id') id): Promise<any> {
      return this.contactsService.delete(id);
    }  
```

Hoàn tất, bước cùng cùng là chúng ta sử dungj PostMan để test những route mà chúng ta vừa định nghĩa:

- Create: [http://127.0.0.1:3000/contacts/create](http://127.0.0.1:3000/contacts/create) - Method: POST
- Get all: [http://127.0.0.1:3000/contacts/](http://127.0.0.1:3000/contacts/) - Method: GET 
- Update: [http://127.0.0.1:3000/contacts/1/update](http://127.0.0.1:3000/contacts/1/update) - Method: PUT
- Delete: [http://127.0.0.1:3000/contacts/1/delete](http://127.0.0.1:3000/contacts/1/delete) - Method: DELETE

Ch
