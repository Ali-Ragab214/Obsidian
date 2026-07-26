تمام. هديك Repository كأنه موجود في مشروع Enterprise، وفيه تقريبًا كل استخدامات JPQL اللي ممكن تقابلها.

نفترض عندنا الـ Entities دي:

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    private String email;

    private Integer age;

    private Double salary;

    private Boolean active;

    @Enumerated(EnumType.STRING)
    private Role role;

    @ManyToOne(fetch = FetchType.LAZY)
    private Department department;
}
```

```java
@Entity
public class Department {

    @Id
    private Long id;

    private String name;
}
```

والـ DTO

```java
public record UserSummaryDto(
        Long id,
        String name,
        String departmentName
) {}
```

---

# UserRepository.java

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // ====================================================
    // Basic Select
    // ====================================================

    @Query("SELECT u FROM User u")
    List<User> findAllUsers();


    // ====================================================
    // WHERE
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE u.email = :email
            """)
    Optional<User> findByEmail(@Param("email") String email);


    @Query("""
            SELECT u
            FROM User u
            WHERE u.age > :age
            """)
    List<User> findUsersOlderThan(@Param("age") int age);


    @Query("""
            SELECT u
            FROM User u
            WHERE u.salary BETWEEN :min AND :max
            """)
    List<User> findSalaryBetween(double min, double max);


    // ====================================================
    // LIKE
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE LOWER(u.name)
            LIKE LOWER(CONCAT('%', :keyword, '%'))
            """)
    List<User> searchByName(String keyword);


    // ====================================================
    // AND
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE u.active = true
            AND u.age >= :age
            """)
    List<User> findActiveAdults(int age);


    // ====================================================
    // OR
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE u.role = :role1
            OR u.role = :role2
            """)
    List<User> findAdminsAndManagers(Role role1, Role role2);


    // ====================================================
    // IN
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE u.role IN :roles
            """)
    List<User> findByRoles(List<Role> roles);


    // ====================================================
    // ORDER BY
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            ORDER BY u.salary DESC
            """)
    List<User> findHighestSalary();


    // ====================================================
    // DISTINCT
    // ====================================================

    @Query("""
            SELECT DISTINCT u.department
            FROM User u
            """)
    List<Department> findDepartments();


    // ====================================================
    // COUNT
    // ====================================================

    @Query("""
            SELECT COUNT(u)
            FROM User u
            WHERE u.active = true
            """)
    Long countActiveUsers();


    // ====================================================
    // SUM
    // ====================================================

    @Query("""
            SELECT SUM(u.salary)
            FROM User u
            """)
    Double totalSalary();


    // ====================================================
    // AVG
    // ====================================================

    @Query("""
            SELECT AVG(u.salary)
            FROM User u
            """)
    Double averageSalary();


    // ====================================================
    // MAX
    // ====================================================

    @Query("""
            SELECT MAX(u.salary)
            FROM User u
            """)
    Double highestSalary();


    // ====================================================
    // MIN
    // ====================================================

    @Query("""
            SELECT MIN(u.salary)
            FROM User u
            """)
    Double lowestSalary();


    // ====================================================
    // INNER JOIN
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            JOIN u.department d
            WHERE d.name = :department
            """)
    List<User> findUsersByDepartment(String department);


    // ====================================================
    // LEFT JOIN
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            LEFT JOIN u.department d
            """)
    List<User> findAllWithDepartment();


    // ====================================================
    // FETCH JOIN
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            JOIN FETCH u.department
            """)
    List<User> fetchUsersWithDepartment();


    // ====================================================
    // DTO Projection
    // ====================================================

    @Query("""
            SELECT new com.example.dto.UserSummaryDto(
                    u.id,
                    u.name,
                    d.name
            )
            FROM User u
            JOIN u.department d
            """)
    List<UserSummaryDto> getUsersSummary();


    // ====================================================
    // Single Field Projection
    // ====================================================

    @Query("""
            SELECT u.email
            FROM User u
            """)
    List<String> getAllEmails();


    // ====================================================
    // GROUP BY
    // ====================================================

    @Query("""
            SELECT d.name,
                   COUNT(u)
            FROM User u
            JOIN u.department d
            GROUP BY d.name
            """)
    List<Object[]> countUsersPerDepartment();


    // ====================================================
    // HAVING
    // ====================================================

    @Query("""
            SELECT d.name,
                   COUNT(u)
            FROM User u
            JOIN u.department d
            GROUP BY d.name
            HAVING COUNT(u) > :count
            """)
    List<Object[]> departmentsHavingMoreThan(long count);


    // ====================================================
    // Sub Query
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE u.salary >
            (
                SELECT AVG(x.salary)
                FROM User x
            )
            """)
    List<User> usersAboveAverageSalary();


    // ====================================================
    // CASE WHEN
    // ====================================================

    @Query("""
            SELECT
            CASE
                WHEN u.salary > 10000
                THEN 'HIGH'
                ELSE 'LOW'
            END
            FROM User u
            """)
    List<String> salaryLevel();


    // ====================================================
    // UPDATE
    // ====================================================

    @Transactional
    @Modifying
    @Query("""
            UPDATE User u
            SET u.active = false
            WHERE u.id = :id
            """)
    int deactivateUser(Long id);


    // ====================================================
    // BULK UPDATE
    // ====================================================

    @Transactional
    @Modifying
    @Query("""
            UPDATE User u
            SET u.salary = u.salary + :bonus
            """)
    int addBonusToEveryone(double bonus);


    // ====================================================
    // DELETE
    // ====================================================

    @Transactional
    @Modifying
    @Query("""
            DELETE
            FROM User u
            WHERE u.active = false
            """)
    int deleteInactiveUsers();


    // ====================================================
    // Pagination
    // ====================================================

    @Query("""
            SELECT u
            FROM User u
            WHERE u.active = true
            """)
    Page<User> findActiveUsers(Pageable pageable);

}
```

---

# في الانترفيو لو اتقالك:

**"إيه أكتر حاجات JPQL بتستخدمها في الشغل؟"**

جاوب بالترتيب ده:

1. ✅ Select with WHERE
    
2. ✅ JOIN / JOIN FETCH
    
3. ✅ DTO Projection (`SELECT new ...`)
    
4. ✅ Aggregation (`COUNT`, `SUM`, `AVG`)
    
5. ✅ Pagination (`Pageable`)
    
6. ✅ Bulk Update/Delete (`@Modifying`)
    
7. ✅ Group By & Having (في الـ Dashboards والـ Reports)
    
8. ✅ Subqueries (زي الموظفين اللي مرتبهم أعلى من المتوسط)
    

---

وده تقريبًا **90% من JPQL** اللي بيتستخدم في مشاريع Spring Boot الحقيقية. الباقي عبارة عن Functions متقدمة أو حالات خاصة، لكن لو فهمت الأمثلة اللي فوق كويس هتكون مستواك قوي جدًا في JPQL.