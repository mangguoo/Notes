## DTO&DAO的区别

> **DTO（Data Transfer Object）** ：**接口边界** 用的数据“形状”（请求/响应）。关注**输入校验、序列化/反序列化** ，不关心数据库。
>
> **DAO（Data Access Object）** ：**持久化层** 的数据访问封装。关注**如何读写数据库** ，不暴露 SQL/ORM 细节给业务层。

```pgsql
Controller  <--->  DTO（校验/转换）
     |
 Service（业务逻辑）
     |
   DAO（读写 DB，封装 ORM/SQL）
     |
 Database
```

### 职责边界


| 维度       | DTO                                     | DAO                         |
| ------------ | ----------------------------------------- | ----------------------------- |
| 关注点     | 接口参数/响应结构                       | 数据库读写封装              |
| 位置       | Controller 边界（入参/出参）            | Service 下层                |
| 技术点     | `class-validator` / `class-transformer` | ORM（TypeORM/Prisma）或 SQL |
| 是否含业务 | 不含（仅描述数据形状+校验）             | 不含（仅数据访问）          |
| 可替换性   | 由接口协议驱动                          | 可以自由切换 ORM/数据源     |
| 与实体关系 | 不等同；可一对多/多对一                 | 直接面向实体/表结构         |

### 常见误区

1. **把实体当 DTO 用** ：会把数据库字段（如 `passwordHash`）暴露到外部；也会把外部传入的字段强行等同数据库结构，耦合严重。
2. **Service 直接操作 Repository** ：短期省事，长期难以替换底层实现；单测也不方便。
3. **DTO 承担业务规则** ：DTO 只负责**形状与校验** ，业务判断放 Service。

## 示例

1. Request/Response DTO (入参/出参)

```ts
// dto/create-user.dto.ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;

  @IsString()
  nickname: string;
}
```

```ts
// dto/user-response.dto.ts
export class UserResponseDto {
  id: string;
  email: string;
  nickname: string;
  createdAt: Date;
}
```

2. Entity（数据库结构）

```ts
// user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn } from 'typeorm';

@Entity('users')
export class UserEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  passwordHash: string;

  @Column()
  nickname: string;

  @CreateDateColumn()
  createdAt: Date;
}
```

3. Mapper（Entity → Response DTO）

```ts
// user.mapper.ts
import { UserEntity } from './user.entity';
import { UserResponseDto } from './dto/user-response.dto';

export class UserMapper {
  static toResponseDto(entity: UserEntity): UserResponseDto {
    const dto = new UserResponseDto();
    dto.id = entity.id;
    dto.email = entity.email;
    dto.nickname = entity.nickname;
    dto.createdAt = entity.createdAt;
    return dto;
  }
}
```

4. DAO（数据访问）

```ts
// user.dao.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { UserEntity } from './user.entity';

@Injectable()
export class UserDao {
  constructor(
    @InjectRepository(UserEntity)
    private readonly repo: Repository<UserEntity>,
  ) {}

  async findByEmail(email: string): Promise<UserEntity | null> {
    return this.repo.findOne({ where: { email } });
  }

  async create(user: Partial<UserEntity>): Promise<UserEntity> {
    const entity = this.repo.create(user);
    return this.repo.save(entity);
  }
}
```

5. Service（编排 + 映射）

```ts
// users.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import * as bcrypt from 'bcryptjs';
import { UserDao } from './user.dao';
import { CreateUserDto } from './dto/create-user.dto';
import { UserResponseDto } from './dto/user-response.dto';
import { UserMapper } from './user.mapper';

@Injectable()
export class UsersService {
  constructor(private readonly userDao: UserDao) {}

  async register(dto: CreateUserDto): Promise<UserResponseDto> {
    const exists = await this.userDao.findByEmail(dto.email);
    if (exists) throw new ConflictException('Email already used');

    const passwordHash = await bcrypt.hash(dto.password, 10);

    const entity = await this.userDao.create({
      email: dto.email,
      passwordHash,
      nickname: dto.nickname,
    });

    return UserMapper.toResponseDto(entity);
  }
}
```

6. Controller（调用 Service，返回 Response DTO）

```ts
// users.controller.ts
import { Body, Controller, Post, UsePipes, ValidationPipe } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UserResponseDto } from './dto/user-response.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post('register')
  @UsePipes(new ValidationPipe({ whitelist: true, transform: true }))
  register(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.register(dto);
  }
}
```
