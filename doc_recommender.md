

# Chức năng gợi ý (Recommendation)

## Tổng quan
Chức năng gợi ý này có lẽ vẫn đang được cải tiến và nghiên cứu, nhưng việc áp dụng nó vào thực tế đã mang lại những hiệu quả nhất định cho website và người dùng. Thánh nhân cầm trịch trong chức năng này là hai bác 石井淳史 (Atsushi Ishii) và 津村泰史 (Yasushi Tsumura).

Vào thời điểm viết bài này, thì chức năng gợi ý của chúng ta phải đi qua 3 thuật toán để có được kết quả. Các bác nào có quan tâm nghiên cứu đến hệ thống gợi ý (Recommender System) thì chắc cũng không lấy gì làm lạ.

1. Bộ lọc cộng tác (Collaborative filtering) => Xin gọi là Logic A

2. Chuỗi Markov (Markov chain) => Gọi là Logic B đi

3. Gọi một đống điều kiện của Solr => Logic C cho nó dễ

Và có lưu ý nhỏ là sẽ có 2 cách để lấy dữ liệu. Từ giờ dữ liệu sẽ được gọi là job nhé các bác. Vì dịch vụ hiện tại là đăng job tìm ứng viên.

```ruby
# Lấy job theo cách bình thường
Recommend::Maker.generate

# Lấy theo cách không bình thường
# Cũng giống cái bình thường thôi nhưng loại bỏ mấy thằng client có câu hỏi phụ
Recommend::Maker.generate_except_additional_questions_clients
```
### Recommend::Maker.generate

Xin được giới thiệu sơ qua về flow của cái này:

Các job sẽ được trích xuất dựa trên thứ tự sau:
1. Các job được lấy từ Logic A đồng thời tồn tại trong Logic B. Nó kiểu như này: *Logic A* **∧** *Logic B* (1)
2. Các job được lấy từ Logic B
3. Các job được lấy từ Logic A
4. Các job được lấy từ Logic C. Bước này chỉ thực hiện khi không lấy được đủ số job mong muốn.

Từ các bước trên chúng ta có được đoạn code trông có vẻ đơn giản như sau:

```ruby
  # lib/src/recommend/maker.rb
  # @return レコメンド生成案件
  def generate
    mar_ids = Recommend::Logic::Markov.recommend_jobs(@seed_job.id, @except_jobs_ids.flatten, @limit*5, 2)
    col_ids = Recommend::Logic::Collaborative.recommend_jobs(@seed_job.id, @except_jobs_ids.flatten, @limit*10)
    # 1. mar_ids ∧ col_ids
    ids = mar_ids & col_ids
    # 2. mar_ids
    ids += (mar_ids - ids)
    # 3. col_ids
    ids += (col_ids - ids) if ids.count < @limit

    # knowlege
    if ids.uniq.count < @limit
      key = "generate:#{@seed_job.id} except:#{@except_jobs_ids.flatten.join(",")}::#{@limit}::remote_prefecture_code:#{@remote_prefecture_code}"
      queries = Recommend::Logic::Knowledge.new.recommend_queries(@seed_job, @remote_prefecture_code)
      ids += Rails.cache.fetch("no:#{CACHE_KEY_NO} #{key}", expires_in: 1.hour) do 
        Recommend::Logic::Knowledge.new.recommend_jobs(@seed_job, queries, default_except_query(@seed_job, @except_jobs_ids + ids), @remote_prefecture_code, @limit+5)
      end

    end

    # entry_enableで削られる可能性があるので多めに取得する
    return jobs(ids[0,@limit+5])[0,@limit]
  end
```

### Recommend::Maker.generate_except_additional_questions_clients

Cái này cũng same same cái trên nhưng có hơi khác một chút là sẽ loại bỏ những job thuộc về client có câu hỏi phụ (additional questions) lúc đăng ký entry. Hàm này theo tiên đoán có lẽ sẽ được thay đổi vào một ngày nào đó (Xem *TODO* của các bác)

```ruby
  # lib/src/recommend/maker.rb
  # TODO 追加質問のある案件を除くためにjobインスタンスの取得が必要。それぞれのrogicでの生成に変更したほうが良い。
  def generate_except_additional_questions_clients
    mar_ids = Recommend::Logic::Markov.recommend_jobs(@seed_job.id, @except_jobs_ids.flatten, @limit*5, 2)
    col_ids = Recommend::Logic::Collaborative.recommend_jobs(@seed_job, @except_jobs_ids.flatten, @limit*10)

    # 1. mar_ids ∧ col_ids
    ids = mar_ids & col_ids
    # 2. mar_ids
    ids += (mar_ids - ids)
    # 3. col_ids
    ids += (col_ids - ids) if ids.count < @limit

    jobs = Job.where(id: ids[0,@limit])
    ids = has_not_additional_questions_job(jobs).map(&:id)
    # knowlege
    if ids.uniq.count < @limit
      key = "generate_except_additional_questions_clients:#{@seed_job.id} except:#{@except_jobs_ids.flatten.join(",")}::#{@limit}::remote_prefecture_code:#{@remote_prefecture_code}"
      queries = Recommend::Logic::Knowledge.new.recommend_queries(@seed_job, @remote_prefecture_code)

      ids += Rails.cache.fetch("no:#{CACHE_KEY_NO} #{key}", expires_in: 1.hour) do 
        Recommend::Logic::Knowledge.new.recommend_jobs(@seed_job, queries, additional_questions_except_query(@seed_job, @except_jobs_ids + ids), @remote_prefecture_code, @limit+5)
      end

    end
```

## Các vị trí sử dụng chức năng gợi ý

| Vị trí | Hàm | Hiển thị tối đa | Ghi chú |
|----------------------------------|-----------------------------------------------------------------|-----------|-------------|
| /jobs/.* | `Recommend::Maker.generate` | 10 |  |
| /entries/new/.* | `Recommend::Maker.generate` | 10 | Pop-up |
| /entries/finish.* | `Recommend::Maker.generate_except_additional_questions_clients` | 10 |  |
| /entries/finish.* | `Recommend::Maker.generate_except_additional_questions_clients` | 10 |  |
| /keep | `Recommend::Maker.generate_except_additional_questions_clients` | 10 |  |
| /top | `Recommend::Maker.generate` | 5 |  |
| <Entry complete> | `Recommend::Maker.generate` | 5 | Mail |
| <詳細ページ離脱防止ポップアップ> | `Recommend::Maker.generate` | 10 | Lại là cái pop-up |
| <応募完了メール> | `JobSimilar` | 10 | Hình như là trang archive |

## Tìm hiểu thuật toán sử dụng

## Job Similarity Search

Khó quá bỏ qua

Xem `job_similar.md`

## Chuỗi Markov - Markov chain

Bỏ qua luôn

Xem `markov.md`

```
User a => job_ids: [42792097, 42791911, 42791911, 42796220, 42796220, 42792779]
job_transition: {42792097=>[42791911], 42791911=>[42796220], 42796220=>[42792779]}
Seek job:
Ex data: 
For job_id 42792097 => data: [42791911, 1.0]
reject_filter
```

## Lọc cộng tác - Collaborative Filter

Xem `collaborative.md`

```
entries_email: 
{42956892=>["test@localhost.jp", "test2@arubaito-ex.jp"],
 42791225=>["phuc_test@example.com"],
 42791231=>["phuc_test@example.com"],
 42791247=>["phuc_test@example.com"],
 42791536=>["phuc_test@example.com"] }

entries_index:
{"222222@gmail.com"=>[27090993],
 "999999@yahoo.co.jp"=>[27121449, 27139038, 1],
 "amino+test1@zigexn.co.jp"=>[27090924] }
 ```


## Conditional Search

Phần này được sử dụng khi đống trên kia tìm ra không đủ số lượng mong muốn
1. Thành phố (city_code), loại công việc tuyển dụng (employment_type_codes) , phương thức nhận lương, occupation , shift, thời gian làm việc, đặc trưng ( 1 phần) *(1), (2)*
2. Cùng thành phố, trạng thái tuyển dụng, phương thức nhận lương, occupation , shift, thời gian làm việc（ngắn hạn）, đặc trưng ( 1 phần) *(1), (2)*
3. Cùng thành phố, trạng thái tuyển dụng, occupation, phương thức nhận lương *(2)*
4. Cùng thành phố, thêm vào đó, occupation , tỉnh lấy từ IP
4. Cùng tỉnh, thêm vào đó, hình thái nhận lương , occupation , hình thái tuyển dụng , tỉnh lấy từ IP *(2)*
5. Cùng tỉnh, thêm vào đó, tỉnh, occupation lấy từ IP

Chú thích:
(1): Đặc trưng (pr_code) chỉ sử dụng 02, 03, 04, 06, 07, 11. Trong sô này, mặc dù chỉ một cái đồng nhất thôi thì cũng tiến hành tập hợp đối tương hiển thị.

(2): Phương thức nhận lương thì trường hợp là lương theo giờ, lương theo ngày sẽ loại ra lương theo tháng . Trường hợp lương theo tháng thì không quan tâm lương theo tháng, theo ngày và theo giờ.

