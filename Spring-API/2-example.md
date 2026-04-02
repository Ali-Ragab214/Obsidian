package com.notification.jpa.controller;

import com.notification.jpa.dto.AuthorDTO;
import com.notification.jpa.dto.AuthorRequest;
import com.notification.jpa.service.AuthorService;
import jakarta.validation.Valid;
import lombok.AllArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/*
 ============================================================
 =                Author REST Controller                    =
 ============================================================
 = مسؤول عن إدارة عمليات الـ Authors (CRUD APIs)            =
 = كل الـ endpoints بتبدأ بـ /api/authors                   =
 ============================================================
*/
@RestController
@AllArgsConstructor
@RequestMapping("/api/authors")
public class AuthorController {

    /*
     --------------------------------------------------------
     | Dependency Injection                                  |
     --------------------------------------------------------
     | الكنترولر بيتعامل مع الـ Service مش مع الداتا بيز     |
     --------------------------------------------------------
    */
    private final AuthorService authorService;

    /*
     ========================================================
     = CREATE AUTHOR                                         =
     = POST /api/authors                                     =
     ========================================================
     = بياخد JSON من الـ request ويعمل Author جديد           =
     = بيرجع:                                                =
     =   - Status: 201 CREATED                               =
     =   - Body: AuthorDTO                                   =
     ========================================================
    */
    @PostMapping
    public ResponseEntity<AuthorDTO> createAuthor(
            @RequestBody @Valid AuthorRequest request) {

        // تحويل JSON → Object + تشغيل validations
        AuthorDTO created = authorService.createAuthor(request);

        // رجوع response مع status 201
        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(created);
    }

    /*
     ========================================================
     = GET ALL AUTHORS                                       =
     = GET /api/authors                                      =
     ========================================================
     = بيرجع كل الـ authors                                  =
     =   - Status: 200 OK                                    =
     =   - Body: List<AuthorDTO>                             =
     ========================================================
    */
    @GetMapping
    public ResponseEntity<List<AuthorDTO>> getAllAuthors() {

        List<AuthorDTO> authors = authorService.getAllAuthors();

        return ResponseEntity.ok(authors);
    }

    /*
     ========================================================
     = GET AUTHOR BY ID                                      =
     = GET /api/authors/{id}                                 =
     ========================================================
     = بياخد id من الـ URL                                   =
     = مثال: /api/authors/5                                  =
     = بيرجع:                                                =
     =   - Status: 200 OK                                    =
     =   - Body: AuthorDTO                                   =
     ========================================================
    */
    @GetMapping("/{id}")
    public ResponseEntity<AuthorDTO> getAuthorById(
            @PathVariable Long id) {

        AuthorDTO author = authorService.getAuthorById(id);

        return ResponseEntity.ok(author);
    }

    /*
     ========================================================
     = UPDATE AUTHOR                                         =
     = PUT /api/authors/{id}                                 =
     ========================================================
     = بيحدث Author موجود                                   =
     = بياخد:                                                =
     =   - id من الـ URL                                     =
     =   - JSON من الـ body                                  =
     = بيرجع:                                                =
     =   - Status: 200 OK                                    =
     =   - Body: AuthorDTO                                   =
     ========================================================
    */
    @PutMapping("/{id}")
    public ResponseEntity<AuthorDTO> updateAuthor(
            @PathVariable Long id,
            @RequestBody @Valid AuthorRequest request) {

        AuthorDTO updated = authorService.updateAuthor(id, request);

        return ResponseEntity.ok(updated);
    }

    /*
     ========================================================
     = DELETE AUTHOR                                         =
     = DELETE /api/authors/{id}                              =
     ========================================================
     = بيحذف Author                                          =
     = بيرجع:                                                =
     =   - Status: 204 NO CONTENT                            =
     =   - Body: none                                        =
     ========================================================
    */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteAuthor(
            @PathVariable Long id) {

        authorService.deleteAuthor(id);

        return ResponseEntity.noContent().build();
    }
}